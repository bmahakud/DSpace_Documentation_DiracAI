package org.dspace.app.rest.diracai.controller;

import jakarta.servlet.http.HttpServletRequest;
import org.apache.pdfbox.pdmodel.common.PDRectangle;
import org.apache.pdfbox.pdmodel.graphics.image.PDImageXObject;
import org.apache.pdfbox.pdmodel.graphics.state.PDExtendedGraphicsState;
import org.dspace.app.rest.diracai.dto.EncryptRequest;
import org.dspace.app.rest.diracai.service.FileAccessLogService;
import org.dspace.app.rest.diracai.util.AESUtil;
import org.dspace.app.rest.diracai.util.FileAccessLogger;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.content.Bitstream;
import org.dspace.content.factory.ContentServiceFactory;
import org.dspace.content.service.BitstreamService;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.dspace.services.RequestService;
import org.dspace.storage.bitstore.service.BitstreamStorageService;
import org.dspace.xoai.services.api.context.ContextService;
import org.dspace.xoai.services.api.context.ContextServiceException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.InputStreamResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.sql.SQLException;


import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.encryption.AccessPermission;
import org.apache.pdfbox.pdmodel.encryption.StandardProtectionPolicy;

import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageTree;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;
import org.apache.pdfbox.util.Matrix;



@CrossOrigin(origins = "http://localhost:5500") // or wherever your HTML is hosted
@RestController
@RequestMapping("/api/diracai")
public class EncryptedBitstreamController {

    @Autowired
    private BitstreamService bitstreamService;

    @Autowired
    private ContentServiceFactory contentServiceFactory;

    @Autowired
    private ContextService contextService;

    @Autowired
    private BitstreamStorageService bitstreamStorageService;

    @Autowired
    private RequestService requestService;

    @Autowired
    private FileAccessLogService fileAccessLogService;

    @Autowired
    private FileAccessLogger fileAccessLogger;

    @Autowired
    private ConfigurationService configurationService;

    private static final Logger log = LoggerFactory.getLogger(EncryptedBitstreamController.class);

//
//    @PostMapping("/encrypt-bitstream")
//    public ResponseEntity<InputStreamResource> encryptAndReturnPdf(
//            @RequestBody EncryptRequest request, HttpServletRequest servletRequest) {
//
//        System.out.println("Received request to encrypt bitstream: " + request.getBitstreamId());
//
//        Context context;
//        try {
//            context = contextService.getContext();
//            System.out.println("DSpace context acquired.");
//        } catch (ContextServiceException e) {
//            System.err.println("Failed to get DSpace context.");
//            e.printStackTrace();
//            throw new RuntimeException(e);
//        }
//
//        Bitstream bitstream;
//        try {
//            bitstream = bitstreamService.findByIdOrLegacyId(context, request.getBitstreamId());
//            if (bitstream != null) {
//                System.out.println("Bitstream found: " + bitstream.getName() + ", ID: " + bitstream.getID());
//            } else {
//                System.err.println("Bitstream not found for ID: " + request.getBitstreamId());
//                return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
//            }
//        } catch (SQLException e) {
//            System.err.println("Error retrieving bitstream.");
//            e.printStackTrace();
//            throw new RuntimeException(e);
//        }
//
//        InputStream originalStream;
//        try {
//            System.out.println("Logging file access...");
//            try {
//                context = ContextUtil.obtainContext(servletRequest);
//                fileAccessLogger.logAccess(context, bitstream.getID(), "DOWNLOAD", servletRequest);
//            } catch (Exception e) {
//                log.error("Failed to log file access", e);
//            }
//            System.out.println("Retrieving bitstream content...");
//            originalStream = bitstreamStorageService.retrieve(context, bitstream);
//            System.out.println("Bitstream retrieved successfully.");
//
//            System.out.println("Encrypting bitstream...");
//            byte[] encryptedBytes = AESUtil.encrypt(originalStream, "your-256-bit-secret-password");
//            System.out.println("Encryption complete. Encrypted bytes length: " + encryptedBytes.length);
//
//            InputStream encryptedStream = new ByteArrayInputStream(encryptedBytes);
//
//            System.out.println("Sending encrypted file as response...");
//            return ResponseEntity.ok()
//                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"encrypted.bin\"")
//                    .contentType(MediaType.APPLICATION_OCTET_STREAM)
//                    .body(new InputStreamResource(encryptedStream));
//
//        } catch (Exception e) {
//            System.err.println("Exception occurred during bitstream encryption or streaming.");
//            e.printStackTrace();
//            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
//        }
//    }


    @PostMapping("/encrypt-bitstream")
    public ResponseEntity<InputStreamResource> encryptAndReturnPdf(@RequestBody EncryptRequest request,
                                                                   HttpServletRequest servletRequest) {
        try {
            System.out.println("Received request to encrypt bitstream: " + request.getBitstreamId());

            Context context = contextService.getContext();
            Bitstream bitstream;
            try {
                bitstream = bitstreamService.findByIdOrLegacyId(context, request.getBitstreamId());
                if (bitstream != null) {
                    System.out.println("Bitstream found: " + bitstream.getName() + ", ID: " + bitstream.getID());
                } else {
                    System.err.println("Bitstream not found for ID: " + request.getBitstreamId());
                    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
                }
            } catch (SQLException e) {
                System.err.println("Error retrieving bitstream.");
                e.printStackTrace();
                throw new RuntimeException(e);
            }

            try {
                context = ContextUtil.obtainContext(servletRequest);
                fileAccessLogger.logAccess(context, bitstream.getID(), "DOWNLOAD", servletRequest);
            } catch (Exception e) {
                log.error("Failed to log file access", e);
            }


            InputStream originalStream = bitstreamStorageService.retrieve(context, bitstream);

            PDDocument document = PDDocument.load(originalStream);

            addImageWatermark(document);

            AccessPermission ap = new AccessPermission();
            ap.setCanPrint(true);
            ap.setCanPrint(false);
            ap.setCanExtractContent(false);
            ap.setCanModify(false);

            String ownerPassword = "adminpass";
            String userPassword = "userpass";

            StandardProtectionPolicy spp = new StandardProtectionPolicy(ownerPassword, userPassword, ap);
            spp.setEncryptionKeyLength(256);
            spp.setPermissions(ap);
            document.protect(spp);

            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            document.save(baos);
            document.close();

            InputStream protectedStream = new ByteArrayInputStream(baos.toByteArray());

            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"protected.pdf\"")
                    .contentType(MediaType.APPLICATION_PDF)
                    .body(new InputStreamResource(protectedStream));

        } catch (Exception e) {
            System.err.println("Error applying password protection to PDF");
            e.printStackTrace();
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
        }
    }






    @PostMapping("/decrypt-bitstream")
    public ResponseEntity<InputStreamResource> decryptUploadedFile(
            @RequestParam("file") MultipartFile file) {

        try {
            System.out.println("Received file to decrypt: " + file.getOriginalFilename());

            byte[] encryptedBytes = file.getBytes();

            System.out.println("Decrypting with AES...");
            byte[] decryptedBytes = AESUtil.decrypt(encryptedBytes, "your-256-bit-secret-password");
            System.out.println("Decryption complete. Bytes: " + decryptedBytes.length);

            InputStream decryptedStream = new ByteArrayInputStream(decryptedBytes);

            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"decrypted.pdf\"")
                    .contentType(MediaType.APPLICATION_PDF)
                    .body(new InputStreamResource(decryptedStream));

        } catch (Exception e) {
            System.err.println("Decryption failed.");
            e.printStackTrace();
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
        }


    }

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

            try (PDPageContentStream contentStream = new PDPageContentStream(document, page, PDPageContentStream.AppendMode.APPEND, true, true)) {
                contentStream.setGraphicsStateParameters(gs);
                contentStream.drawImage(pdImage, x, y, imageWidth, imageHeight);
            }
        }
    }
}
