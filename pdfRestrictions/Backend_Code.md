
package org.dspace.app.rest.diracai.controller;

import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.common.PDRectangle;
import org.apache.pdfbox.pdmodel.graphics.image.PDImageXObject;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.PDPageContentStream.AppendMode;
import org.apache.pdfbox.pdmodel.graphics.state.PDExtendedGraphicsState;
import org.dspace.app.rest.diracai.service.FileAccessLogService;
import org.dspace.app.rest.diracai.util.FileAccessLogger;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.authorize.ResourcePolicy;
import org.dspace.authorize.service.AuthorizeService;
import org.dspace.authorize.service.ResourcePolicyService;
import org.dspace.content.Bitstream;
import org.dspace.content.service.BitstreamService;
import org.dspace.core.Context;
import org.dspace.eperson.EPerson;
import org.dspace.eperson.service.GroupService;
import org.dspace.services.ConfigurationService;
import org.dspace.services.RequestService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/custom/bitstreams")
public class RestrictedPdfController {

    @Autowired
    private BitstreamService bitstreamService;

    @Autowired
    private ResourcePolicyService resourcePolicyService;

    @Autowired
    private RequestService requestService;

    @Autowired
    private AuthorizeService authorizeService;

    @Autowired
    private GroupService groupService;

    @Autowired
    private FileAccessLogService fileAccessLogService;

    @Autowired
    private FileAccessLogger fileAccessLogger;

    @Autowired
    private ConfigurationService configurationService;

    private static final Logger log = LoggerFactory.getLogger(RestrictedPdfController.class);

    @GetMapping("/{uuid}/filtered-content")
    public void getFilteredContent(@PathVariable UUID uuid,
                                   HttpServletRequest request,
                                   HttpServletResponse response) throws Exception {

        Context context = ContextUtil.obtainContext(request);
        Bitstream bitstream = bitstreamService.findByIdOrLegacyId(context, uuid.toString());

        // Log access
        try {
            fileAccessLogger.logAccess(context, uuid, "VIEW", request);
        } catch (Exception e) {
            log.error("Failed to log file access", e);
        }

        if (bitstream == null) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND, "Bitstream not found.");
            return;
        }

        EPerson currentUser = context.getCurrentUser();

        // --- CASE 1: Admin gets full PDF ---
        if (currentUser != null && authorizeService.isAdmin(context, currentUser)) {
            try (InputStream is = bitstreamService.retrieve(context, bitstream);
                 PDDocument doc = PDDocument.load(is)) {
                response.setContentType("application/pdf");
                addImageWatermark(doc);
                doc.save(response.getOutputStream());
            } catch (IOException e) {
                response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, "Unable to read the PDF content.");
            }
            return;
        }

        // Get all policies for the bitstream
        List<ResourcePolicy> allPolicies = resourcePolicyService.find(context, bitstream);

        // Match user/group/anonymous policies
        List<ResourcePolicy> matchedPolicies = allPolicies.stream()
                .filter(policy -> {
                    try {
                        return (currentUser != null && policy.getEPerson() != null && policy.getEPerson().equals(currentUser)) ||
                                (policy.getGroup() != null &&
                                        ("Anonymous".equals(policy.getGroup().getName()) ||
                                                (currentUser != null && groupService.isMember(context, currentUser, policy.getGroup())))
                                );
                    } catch (SQLException e) {
                        throw new RuntimeException(e);
                    }
                })
                .collect(Collectors.toList());

        // --- CASE 2: No matching policies => full PDF ---
        if (matchedPolicies.isEmpty()) {
            try (InputStream is = bitstreamService.retrieve(context, bitstream);
                 PDDocument doc = PDDocument.load(is)) {
                response.setContentType("application/pdf");
                addImageWatermark(doc);
                doc.save(response.getOutputStream());
            } catch (IOException e) {
                response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, "Unable to read the PDF content.");
            }
            return;
        }

        // Collect allowed page ranges
        List<int[]> allowedRanges = new ArrayList<>();
        for (ResourcePolicy policy : matchedPolicies) {
            Integer start = policy.getPageStart();
            Integer end = policy.getPageEnd();
            if (start != null && end != null && start > 0 && end >= start) {
                allowedRanges.add(new int[]{start, end});
            }
        }

        // --- CASE 3: Filtered pages based on allowed ranges ---
        try (InputStream is = bitstreamService.retrieve(context, bitstream);
             PDDocument original = PDDocument.load(is);
             PDDocument filtered = new PDDocument()) {

            int total = original.getNumberOfPages();

            for (int i = 0; i < total; i++) {
                int pageNum = i + 1;
                boolean include = false;
                for (int[] range : allowedRanges) {
                    if (pageNum >= range[0] && pageNum <= range[1]) {
                        include = true;
                        break;
                    }
                }
                if (include) {
                    filtered.addPage(original.getPage(i));
                }
            }

            response.setContentType("application/pdf");
            addImageWatermark(filtered);
            filtered.save(response.getOutputStream());

        } catch (IOException e) {
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR, "Unable to process PDF for page-level access.");
        }
    }

    /**
     * Adds a watermark image (if exists in dspace.dir/watermark) to all pages.
     */
    private void addImageWatermark(PDDocument document) throws IOException {
        String dspaceDir = configurationService.getProperty("dspace.dir");
        File watermarkFolder = new File(dspaceDir, "watermark");

        if (!watermarkFolder.exists() || !watermarkFolder.isDirectory()) {
            log.warn("Watermark folder not found: {}", watermarkFolder.getAbsolutePath());
            return;
        }

        File[] files = watermarkFolder.listFiles((dir, name) -> name.toLowerCase().startsWith("image."));
        if (files == null || files.length == 0) {
            log.warn("No watermark image found in: {}", watermarkFolder.getAbsolutePath());
            return;
        }

        PDImageXObject pdImage = PDImageXObject.createFromFileByContent(files[0], document);

        for (PDPage page : document.getPages()) {
            PDRectangle pageSize = page.getMediaBox();

            // Scale watermark to 40% page width
            float scale = (pageSize.getWidth() * 0.4f) / pdImage.getWidth();
            float imageWidth = pdImage.getWidth() * scale;
            float imageHeight = pdImage.getHeight() * scale;

            // Center position
            float x = (pageSize.getWidth() - imageWidth) / 2;
            float y = (pageSize.getHeight() - imageHeight) / 2;

            // Transparency
            PDExtendedGraphicsState gs = new PDExtendedGraphicsState();
            gs.setNonStrokingAlphaConstant(0.3f); // 30% opacity

            try (PDPageContentStream contentStream = new PDPageContentStream(document, page, AppendMode.APPEND, true, true)) {
                contentStream.setGraphicsStateParameters(gs);
                contentStream.drawImage(pdImage, x, y, imageWidth, imageHeight);
            }
        }
    }
}
