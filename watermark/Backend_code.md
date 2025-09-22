Location of the file : dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller
name of the file : WatermarkController.java



package org.dspace.app.rest.diracai.controller;


import org.dspace.services.ConfigurationService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.*;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.Map;

@RestController
@RequestMapping("/api/watermark")
public class WatermarkController {

    @Autowired
    private ConfigurationService configurationService;

    @GetMapping
    public ResponseEntity<byte[]> getCurrent() {
        String dspaceDir = configurationService.getProperty("dspace.dir");
        File imageFolder = new File(dspaceDir, "watermark");

        if (!imageFolder.exists() || !imageFolder.isDirectory()) {
            return ResponseEntity.notFound().build();
        }

        // Find the first file in the folder (assuming only one watermark exists)
        File[] files = imageFolder.listFiles((dir, name) -> name.toLowerCase().startsWith("image."));
        if (files == null || files.length == 0) {
            return ResponseEntity.noContent().build();
        }

        File imageFile = files[0];
        try {
            byte[] fileContent = java.nio.file.Files.readAllBytes(imageFile.toPath());
            String contentType = java.nio.file.Files.probeContentType(imageFile.toPath());

            return ResponseEntity.ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "inline; filename=\"" + imageFile.getName() + "\"")
                    .contentType(MediaType.parseMediaType(contentType != null ? contentType : "application/octet-stream"))
                    .body(fileContent);
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }


    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @PreAuthorize("hasAuthority('ADMIN') or hasAuthority('SYSTEM_ADMIN')") // adjust to your roles
    public ResponseEntity<?> upload(@RequestParam("file") MultipartFile file) {
        try {
            if (file == null || file.isEmpty()) {
                return ResponseEntity.badRequest().body("Missing file");
            }

            // Get dspace.dir and watermark folder
            String dspaceDir = configurationService.getProperty("dspace.dir");
            File watermarkFolder = new File(dspaceDir, "watermark");

            if (!watermarkFolder.exists()) {
                watermarkFolder.mkdirs();
            }

            // ✅ Delete any existing watermark file regardless of extension
            File[] existingFiles = watermarkFolder.listFiles((dir, name) -> name.toLowerCase().startsWith("image."));
            if (existingFiles != null) {
                for (File f : existingFiles) {
                    if (!f.delete()) {
                        System.out.println("bye pooja");
                    }
                }
            }

            // Keep name "image" with original extension
            String originalFilename = file.getOriginalFilename();
            String extension = "";
            if (originalFilename != null && originalFilename.contains(".")) {
                extension = originalFilename.substring(originalFilename.lastIndexOf("."));
            }
            File targetFile = new File(watermarkFolder, "image" + extension);

            file.transferTo(targetFile);

            return ResponseEntity.status(HttpStatus.CREATED)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(Map.of(
                            "filename", targetFile.getName(),
                            "contentType", file.getContentType(),
                            "message", "Watermark uploaded successfully"
                    ));

        } catch (IOException ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Failed to store watermark: " + ex.getMessage());
        }
    }


}

