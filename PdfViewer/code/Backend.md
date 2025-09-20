package org.dspace.app.rest.diracai.controller;
//
//import jakarta.servlet.http.HttpServletRequest;
//import org.dspace.app.rest.utils.ContextUtil;
//import org.dspace.authorize.ResourcePolicy;
//import org.dspace.authorize.service.AuthorizeService;
//import org.dspace.authorize.service.ResourcePolicyService;
//import org.dspace.content.Bitstream;
//import org.dspace.content.service.BitstreamService;
//import org.dspace.core.Constants;
//import org.dspace.core.Context;
//import org.dspace.eperson.EPerson;
//import org.dspace.services.RequestService;
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.http.ResponseEntity;
//import org.springframework.web.bind.annotation.*;
//
//import java.sql.SQLException;
//import java.text.SimpleDateFormat;
//import java.util.*;
//import java.util.stream.Collectors;
//import org.dspace.core.Context;
//import org.dspace.eperson.EPerson;
//import org.dspace.eperson.Group;
//import org.dspace.eperson.service.GroupService;
//import org.dspace.authorize.ResourcePolicy;
//import org.dspace.authorize.service.AuthorizeService;
//import org.dspace.authorize.service.ResourcePolicyService;
//import org.dspace.content.Bitstream;
//import org.dspace.content.service.BitstreamService;
//import org.dspace.app.rest.utils.ContextUtil;
//
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.http.ResponseEntity;
//import org.springframework.web.bind.annotation.GetMapping;
//import org.springframework.web.bind.annotation.PathVariable;
//import org.springframework.web.bind.annotation.RequestMapping;
//import org.springframework.web.bind.annotation.RestController;
//
//import jakarta.servlet.http.HttpServletRequest;
//
//import java.text.SimpleDateFormat;
//import java.util.*;
//import java.util.stream.Collectors;
//
//@RestController
//@RequestMapping("/api/custom/bitstreams")
//public class BitstreamPermissionController {
//
//    @Autowired
//    private BitstreamService bitstreamService;
//
//    @Autowired
//    private ResourcePolicyService resourcePolicyService;
//
//    @Autowired
//    private RequestService requestService;
//
//    @Autowired
//    private AuthorizeService authorizeService;
//
//    @Autowired
//    private GroupService groupService;
//
//
//    @GetMapping("/{uuid}/permissions")
//    public ResponseEntity<?> getUserBitstreamPermissions(@PathVariable UUID uuid,
//                                                         HttpServletRequest request) throws Exception {
//
//        Context context = ContextUtil.obtainContext(request);
//        Bitstream bitstream = bitstreamService.findByIdOrLegacyId(context, uuid.toString());
//        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//
//        if (bitstream == null) {
//            return ResponseEntity.notFound().build();
//        }
//
//        EPerson currentUser = context.getCurrentUser();
//        if (currentUser == null) {
//            return ResponseEntity.status(401).body("Unauthorized");
//        }
//
//        boolean isAdmin = authorizeService.isAdmin(context, currentUser);
//
//        List<ResourcePolicy> allPolicies = resourcePolicyService.find(context, bitstream);
//        Date now = new Date();
//
//        List<Map<String, Object>> userPolicies = allPolicies.stream()
//                .filter(policy ->
//                        {
//                            try {
//                                return ((policy.getEPerson() != null && policy.getEPerson().equals(currentUser)) ||
//                                        (policy.getGroup() != null &&
//                                                ("Anonymous".equals(policy.getGroup().getName()) ||
//                                                        groupService.isMember(context, currentUser, policy.getGroup()))
//                                        ))
//                                        &&
//                                        (policy.getStartDate() == null || !now.before(policy.getStartDate()))
//                                        &&
//                                        (policy.getEndDate() == null || !now.after(policy.getEndDate()));
//                            } catch (SQLException e) {
//                                throw new RuntimeException(e);
//                            }
//                        }
//                )
//                .map(policy -> {
//                    Map<String, Object> map = new HashMap<>();
//                    map.put("name", policy.getRpName());
//                    map.put("description", policy.getRpDescription());
//                    map.put("policyType", policy.getRpType());
//                    map.put("action", resolveActionName(policy.getAction()));
//                    map.put("startDate", policy.getStartDate() != null ? formatter.format(policy.getStartDate()) : null);
//                    map.put("endDate", policy.getEndDate() != null ? formatter.format(policy.getEndDate()) : null);
//                    map.put("pageStart", policy.getPageStart());
//                    map.put("pageEnd", policy.getPageEnd());
//                    map.put("print", policy.isPrint());
//                    map.put("download", policy.isDownload());
//                    return map;
//                })
//                .collect(Collectors.toList());
//
//        return ResponseEntity.ok(Map.of(
//                "bitstreamId", uuid.toString(),
//                "userId", currentUser.getID(),
//                "isAdmin", isAdmin,
//                "policies", userPolicies
//        ));
//    }
//
//
//
//
//    private String resolveActionName(int actionId) {
//        return Constants.actionText[actionId];
//    }
//}



import jakarta.servlet.http.HttpServletRequest;
import org.dspace.app.rest.diracai.service.PdfaConverter;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.authorize.ResourcePolicy;
import org.dspace.authorize.service.AuthorizeService;
import org.dspace.authorize.service.ResourcePolicyService;
import org.dspace.content.Bitstream;
import org.dspace.content.Item;
import org.dspace.content.MetadataValue;
import org.dspace.content.service.BitstreamService;
import org.dspace.content.service.ItemService;
import org.dspace.core.Constants;
import org.dspace.core.Context;
import org.dspace.eperson.EPerson;
import org.dspace.services.RequestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.io.*;
import java.sql.SQLException;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;
import org.dspace.core.Context;
import org.dspace.eperson.EPerson;
import org.dspace.eperson.Group;
import org.dspace.eperson.service.GroupService;
import org.dspace.authorize.ResourcePolicy;
import org.dspace.authorize.service.AuthorizeService;
import org.dspace.authorize.service.ResourcePolicyService;
import org.dspace.content.Bitstream;
import org.dspace.content.service.BitstreamService;
import org.dspace.app.rest.utils.ContextUtil;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import jakarta.servlet.http.HttpServletRequest;

import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;

import static org.dspace.app.rest.diracai.service.PdfaConverter.convertToPdfA;




//
//import jakarta.servlet.http.HttpServletRequest;
//import org.dspace.app.rest.diracai.handler.BitstreamPermissionHandler;
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.http.ResponseEntity;
//import org.springframework.web.bind.annotation.GetMapping;
//import org.springframework.web.bind.annotation.PathVariable;
//import org.springframework.web.bind.annotation.RequestMapping;
//import org.springframework.web.bind.annotation.RestController;
//
//import java.util.UUID;
//
//@RestController
//@RequestMapping("/api/custom/bitstreams")
//public class BitstreamPermissionController {
//
//    @Autowired
//    private BitstreamPermissionHandler handler;
//
//    @GetMapping("/{uuid}/permissions")
//    public ResponseEntity<?> getUserBitstreamPermissions(@PathVariable UUID uuid, HttpServletRequest request) {
//        return ResponseEntity.ok(handler.handle(uuid, request));
//    }
//}



@RestController
@RequestMapping("/api/custom/bitstreams")
public class BitstreamPermissionController {

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
    private ItemService itemService;

    @GetMapping("/{uuid}/permissions")
    public ResponseEntity<?> getUserBitstreamPermissions(@PathVariable UUID uuid,
                                                         HttpServletRequest request) throws Exception {

        Context context = ContextUtil.obtainContext(request);
                context.turnOffAuthorisationSystem();

        Bitstream bitstream = bitstreamService.findByIdOrLegacyId(context, uuid.toString());
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        if (bitstream == null) {
            return ResponseEntity.notFound().build();
        }

        EPerson currentUser = context.getCurrentUser();
        if (currentUser == null) {
            return ResponseEntity.status(401).body("Unauthorized");
        }



        String mimeType = bitstream.getFormat(context).getMIMEType();
        boolean isPdf = "application/pdf".equalsIgnoreCase(mimeType);

        Item owningItem = bitstream.getBundles().stream()
                .flatMap(bundle -> bundle.getItems().stream())
                .findFirst()
                .orElse(null);

        if (owningItem == null) {
            return ResponseEntity.status(404).body("Owning item not found");
        }

        final String METADATA_SCHEMA = "dc";
        final String METADATA_ELEMENT = "date";
        final String METADATA_QUALIFIER = "disposal";

        List<MetadataValue> metadataValues = itemService.getMetadata(
                owningItem, METADATA_SCHEMA, METADATA_ELEMENT, METADATA_QUALIFIER, Item.ANY);

        Date disposalDate = null;
        if (metadataValues != null && !metadataValues.isEmpty()) {
            try {
                disposalDate = new SimpleDateFormat("yyyy-MM-dd").parse(metadataValues.get(0).getValue());
            } catch (Exception e) {
                return ResponseEntity.status(400).body("Invalid disposal date format");
            }
        }

        File inputFile = null;
        File pdfaFile = null;
        boolean pdfaConverted = false; // ✅ Track PDF/A conversion status
        Date now = new Date();

        if (isPdf && disposalDate != null && now.after(disposalDate)) {
            try {
                inputFile = File.createTempFile("original-", ".pdf");
                try (InputStream input = bitstreamService.retrieve(context, bitstream);
                     OutputStream output = new FileOutputStream(inputFile)) {
                    input.transferTo(output);
                    pdfaFile = PdfaConverter.convertToPdfA(inputFile);
                    if (pdfaFile == null || !pdfaFile.exists()) {
                        return ResponseEntity.status(500).body("PDF/A conversion failed");
                    }
                    File savedCopy = new File("/home/dspace/dspace/July_7th/test/pdfa-" + uuid + ".pdf");
                    try (InputStream pdfaInput = new FileInputStream(pdfaFile);
                         OutputStream savedOut = new FileOutputStream(savedCopy)) {
                        pdfaInput.transferTo(savedOut);
                    }

                    replaceBitstreamContent(context, bitstream, pdfaFile);

                    pdfaConverted = true;
                }
            } catch (Exception e) {
                return ResponseEntity.status(500).body("Error during PDF/A conversion: " + e.getMessage());
            } finally {
                if (inputFile != null && inputFile.exists()) inputFile.delete();
                if (pdfaFile != null && pdfaFile.exists()) pdfaFile.delete();
                                context.restoreAuthSystemState();

            }
        }

        boolean isAdmin = authorizeService.isAdmin(context, currentUser);

        List<ResourcePolicy> allPolicies = resourcePolicyService.find(context, bitstream);

        List<Map<String, Object>> userPolicies = allPolicies.stream()
                .filter(policy -> {
                    try {
                        return ((policy.getEPerson() != null && policy.getEPerson().equals(currentUser)) ||
                                (policy.getGroup() != null &&
                                        ("Anonymous".equals(policy.getGroup().getName()) ||
                                                groupService.isMember(context, currentUser, policy.getGroup()))))
                                &&
                                (policy.getStartDate() == null || !now.before(policy.getStartDate()))
                                &&
                                (policy.getEndDate() == null || !now.after(policy.getEndDate()));
                    } catch (SQLException e) {
                        throw new RuntimeException(e);
                    }
                })
                .map(policy -> {
                    Map<String, Object> map = new HashMap<>();
                    map.put("name", policy.getRpName());
                    map.put("description", policy.getRpDescription());
                    map.put("policyType", policy.getRpType());
                    map.put("action", resolveActionName(policy.getAction()));
                    map.put("startDate", policy.getStartDate() != null ? formatter.format(policy.getStartDate()) : null);
                    map.put("endDate", policy.getEndDate() != null ? formatter.format(policy.getEndDate()) : null);
                    map.put("pageStart", policy.getPageStart());
                    map.put("pageEnd", policy.getPageEnd());
                    map.put("print", policy.isPrint());
                    map.put("download", policy.isDownload());
                    return map;
                })
                .collect(Collectors.toList());

        return ResponseEntity.ok(Map.of(
                "bitstreamId", uuid.toString(),
                "userId", currentUser.getID(),
                "isAdmin", isAdmin,
                "pdfaConverted", pdfaConverted,
                "policies", userPolicies
        ));
    }





    private String resolveActionName(int actionId) {
        return Constants.actionText[actionId];
    }

    private void replaceBitstreamContent(Context context, Bitstream bitstream, File newFile) throws Exception {
        try (InputStream newStream = new FileInputStream(newFile)) {
            bitstreamService.updateContents(context, bitstream, newStream);
            bitstream.setSizeBytes(newFile.length());
            bitstreamService.setFormat(context, bitstream, bitstream.getFormat(context));
            bitstreamService.update(context, bitstream);
            context.commit();
        }
    }

}

