<!-- //package org.dspace.app.rest.diracai.controller;
//
//import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
//import org.dspace.app.rest.diracai.service.FileHashService;
//import org.dspace.content.Diracai.FileHashRecord;
//import org.dspace.core.Context;
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.data.domain.Page;
//import org.springframework.data.domain.PageRequest;
//import org.springframework.data.domain.Pageable;
//import org.springframework.data.domain.Sort;
//import org.springframework.http.HttpStatus;
//import org.springframework.http.ResponseEntity;
//import org.springframework.web.bind.annotation.*;
//
//import java.io.IOException;
//import java.util.ArrayList;
//import java.util.List;
//
//@RestController
//@RequestMapping("/api/cnr")
//public class FileHashController {
//
//    @Autowired
//    private FileHashService fileHashService;
//
//    @Autowired
//    private FileHashRecordRepository recordRepository;
//
//    // POST: Generate ZIP + SHA256 + Store
//    @PostMapping("/generate")
//    public FileHashRecord generateAndStoreHash(@RequestParam("cnr") String cnr,@RequestParam("docType") String docType, Context context) throws IOException {
//        return fileHashService.generateZipAndHash(cnr, context,docType);
//    }
//
//    @GetMapping("/records")
//    public ResponseEntity<Page<FileHashRecord>> getAllHashes(
//            @RequestParam(defaultValue = "0") int page,
//            @RequestParam(defaultValue = "10") int size,
//            @RequestParam(defaultValue = "createdAt") String sortBy,
//            @RequestParam(defaultValue = "desc") String sortDir,
//            @RequestParam(required = false) String submitted   // NEW PARAM
//    ) {
//        try {
//            Sort sort = sortDir.equalsIgnoreCase("asc")
//                    ? Sort.by(sortBy).ascending()
//                    : Sort.by(sortBy).descending();
//
//            Pageable pageable = PageRequest.of(page, size, sort);
//            Page<FileHashRecord> result;
//
//            if ("submit".equalsIgnoreCase(submitted)) {
//                result = recordRepository.findByAckIdIsNotNullAndAckIdNot(pageable, "");
//            } else if ("notSubmitted".equalsIgnoreCase(submitted)) {
//                result = recordRepository.findByAckIdIsNullOrAckId(pageable, "");
//            } else {
//                result = recordRepository.findAll(pageable);
//            }
//
//            return ResponseEntity.ok(result);
//        } catch (Exception e) {
//            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
//        }
//    }
//
//
//}



package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.app.rest.diracai.service.FileHashService;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.core.Context;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;

@RestController
@RequestMapping("/api/cnr")
public class FileHashController {

    @Autowired
    private FileHashService fileHashService;

    @Autowired
    private FileHashRecordRepository recordRepository;

    // POST: Generate ZIP + SHA256 + Store
    @PostMapping("/generate")
    public FileHashRecord generateAndStoreHash(@RequestParam("cnr") String cnr,
                                               @RequestParam("docType") String docType,
                                               Context context) throws IOException {
        return fileHashService.generateZipAndHash(cnr, context, docType);
    }

    @GetMapping("/records")
    public ResponseEntity<Page<FileHashRecord>> getAllHashes(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "createdAt") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir,
            @RequestParam(required = false) String submitted
    ) {
        try {
            Sort sort = sortDir.equalsIgnoreCase("asc")
                    ? Sort.by(sortBy).ascending()
                    : Sort.by(sortBy).descending();

            Pageable pageable = PageRequest.of(page, size, sort);
            Page<FileHashRecord> result;

            if ("submit".equalsIgnoreCase(submitted)) {
                result = recordRepository.findByAckIdIsNotNullAndAckIdNot(pageable, "");
            } else if ("notSubmitted".equalsIgnoreCase(submitted)) {
                result = recordRepository.findByAckIdIsNullOrAckId(pageable, "");
            } else {
                result = recordRepository.findAll(pageable);
            }

            return ResponseEntity.ok(result);
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    /** DELETE: remove the generated zip and its DB record by fileName */
    @DeleteMapping("/records/{fileName}")
    public ResponseEntity<Void> deleteGeneratedRecord(@PathVariable String fileName) {
        try {
            FileHashService.DeleteResult result = fileHashService.deleteZipAndRecord(fileName);
            if (result == FileHashService.DeleteResult.NOT_FOUND) {
                return ResponseEntity.notFound().build();
            }
            return ResponseEntity.noContent().build(); // 204
        } catch (IOException ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    /** Optional alias: DELETE /api/cnr/zip/{fileName} */
    @DeleteMapping("/zip/{fileName}")
    public ResponseEntity<Void> deleteGeneratedZip(@PathVariable String fileName) {
        try {
            FileHashService.DeleteResult result = fileHashService.deleteZipAndRecord(fileName);
            if (result == FileHashService.DeleteResult.NOT_FOUND) {
                return ResponseEntity.notFound().build();
            }
            return ResponseEntity.noContent().build();
        } catch (IOException ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
}










package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.FileHashRecord;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;
import java.util.List;

public interface FileHashRecordRepository extends JpaRepository<FileHashRecord, Long> {
    FileHashRecord findByFileName(String cnr);
    FileHashRecord findByAckId(String ackId);
    List<FileHashRecord> findAll(Sort sort);
    @Query("""
        select f from FileHashRecord f
        where f.createdAt between :from and :to
        order by f.createdAt desc
    """)
    List<FileHashRecord> findAllForReport(@Param("from") LocalDateTime from,
                                          @Param("to") LocalDateTime to);
    Page<FileHashRecord> findByAckIdIsNotNullAndAckIdNot(Pageable pageable, String emptyValue);
    Page<FileHashRecord> findByAckIdIsNullOrAckId(Pageable pageable, String emptyValue);
}






package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "file_hash_record")
public class FileHashRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String fileName;
    private String hashValue;
    private LocalDateTime createdAt;
    private String ackId;
    private String zipStatus;
    private String postResponse;
    private String postStatus;
    private String getCheckResponse;
    private String getCheckStatus;
    private Integer fileCount;
    private LocalDateTime uploadDate;
    private String batchName;
    private String caseType;
    private String caseNo;
    private String Status;
    private String cinoNumber;
    private String createdBy;
    private String uploadedBy;

    public String getCreatedBy() {
        return createdBy;
    }

    public String getUploadedBy() {
        return uploadedBy;
    }

    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }

    public void setUploadedBy(String uploadedBy) {
        this.uploadedBy = uploadedBy;
    }

    public String getCinoNumber() {
        return cinoNumber;
    }

    public void setCinoNumber(String cinoNumber) {
        this.cinoNumber = cinoNumber;
    }

    public String getStatus() {
        return Status;
    }

    public void setStatus(String status) {
        Status = status;
    }

    public void setUploadDate(LocalDateTime uploadDate) {
        this.uploadDate = uploadDate;
    }

    public void setFileCount(Integer fileCount) {
        this.fileCount = fileCount;
    }

    public LocalDateTime getUploadDate() {
        return uploadDate;
    }

    public void setCaseType(String caseType) {
        this.caseType = caseType;
    }

    public void setBatchName(String batchName) {
        this.batchName = batchName;
    }

    public String getCaseType() {
        return caseType;
    }

    public String getCaseNo() {
        return caseNo;
    }

    public String getBatchName() {
        return batchName;
    }

    public void setCaseNo(String caseNo) {
        this.caseNo = caseNo;
    }

    public Integer getFileCount() {
        return fileCount;
    }

    public void setFileCount(int fileCount) {
        this.fileCount = fileCount;
    }

    @PrePersist
    public void onCreate() {
        this.createdAt =  LocalDateTime.now();
    }


    public String getZipStatus() {
        return zipStatus;
    }

    public String getGetCheckResponse() {
        return getCheckResponse;
    }

    public String getGetCheckStatus() {
        return getCheckStatus;
    }

    public String getPostResponse() {
        return postResponse;
    }

    public String getPostStatus() {
        return postStatus;
    }

    public void setGetCheckResponse(String getCheckResponse) {
        this.getCheckResponse = getCheckResponse;
    }

    public void setGetCheckStatus(String getCheckStatus) {
        this.getCheckStatus = getCheckStatus;
    }

    public void setPostResponse(String postResponse) {
        this.postResponse = postResponse;
    }

    public void setPostStatus(String postStatus) {
        this.postStatus = postStatus;
    }

    public void setZipStatus(String zipStatus) {
        this.zipStatus = zipStatus;
    }

    public FileHashRecord() {}


    // Getters and setters

    public void setAckId(String ackId) {
        this.ackId = ackId;
    }

    public String getAckId() {
        return ackId;
    }

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getFileName() { return fileName; }

    public void setFileName(String fileName) { this.fileName = fileName; }

    public String getHashValue() { return hashValue; }

    public void setHashValue(String hashValue) { this.hashValue = hashValue; }

    public LocalDateTime getCreatedAt() { return createdAt; }

    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}


//package org.dspace.app.rest.diracai.controller;
//
//import jakarta.servlet.http.HttpServletResponse;
//import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
//import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
//import org.dspace.core.Context;
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.format.annotation.DateTimeFormat;
//import org.springframework.http.ResponseEntity;
//import org.springframework.web.bind.annotation.*;
//import org.dspace.app.rest.utils.ContextUtil;
//
//import java.io.IOException;
//import java.io.PrintWriter;
//import java.time.LocalDate;
//import java.time.LocalDateTime;
//import java.util.List;
//import java.util.Map;
//
//@RestController
//@RequestMapping("/api/jtdr")
//public class JtdrIntegrationController {
//
//    @Autowired
//    private JtdrIntegrationService jtdrService;
//
//    /**
//     * Submit a case to JTDR using CNR and ZIP hash. No ZIP upload needed; ZIP will be read from local disk.
//     */
//    @PostMapping("/submit")
//    public Map<String, Object> submitCase(
//            @RequestParam("cnr") String cnr) {
//        Context context = ContextUtil.obtainCurrentRequestContext();
//        ;
//        return jtdrService.submitCase(context,cnr);
//    }
//
//    /**
//     * Check status of submitted case using JTDR Acknowledgement ID.
//     */
//
//    @GetMapping("/status/{ackId}")
//    public Map<String, Object> getStatus(@PathVariable String ackId) {
//        return jtdrService.checkStatus(ackId);
//    }
//
//    @GetMapping("/report")
//    public void report(
//            @RequestParam("start") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
//            @RequestParam("end")   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end,
//            HttpServletResponse response
//    ) throws IOException {
//        LocalDateTime from = start.atStartOfDay();
//        LocalDateTime to   = end.atTime(23, 59, 59, 999_000_000);
//
//        List<JtdrDetailedReportRow> rows = jtdrService.getDetailedReport(from, to);
//
//        // Set response headers
//        response.setContentType("text/csv");
//        response.setHeader("Content-Disposition", "attachment; filename=detailed-report.csv");
//
//        // Write CSV directly to output stream
//        try (PrintWriter writer = response.getWriter()) {
//            // Header
//            writer.println("Sl No,Batch Name,Case Type,Case No,Upload Date,Upload Status,Zip Created At,Zip Created By,File Submitted Vy,Zip Status");
//
//            // Rows
//            for (JtdrDetailedReportRow row : rows) {
//                writer.print(row.slNumber); writer.print(",");
//                writer.print(safe(row.batchName)); writer.print(",");
//                writer.print(safe(row.caseType)); writer.print(",");
//                writer.print(safe(row.caseNo)); writer.print(",");
//                writer.print(row.uploadDate != null ? row.uploadDate : ""); writer.print(",");
//                writer.print(safe(row.uploadStatus)); writer.print(",");
//                writer.print(row.zipCreatedAt != null ? row.zipCreatedAt : ""); writer.print(",");
//                writer.print(row.createdBy); writer.print(",");
//                writer.print(row.uploadedBy); writer.print(",");
//                writer.print(safe(row.zipStatus));
//                writer.println();
//            }
//        }
//    }
//
//    private String safe(Object value) {
//        if (value == null) return "";
//        String str = value.toString();
//        // Escape commas/quotes by wrapping in quotes
//        if (str.contains(",") || str.contains("\"")) {
//            str = "\"" + str.replace("\"", "\"\"") + "\"";
//        }
//        return str;
//    }
//
//}


package org.dspace.app.rest.diracai.controller;

import jakarta.servlet.http.HttpServletResponse;
import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;

// PDFBox
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.common.PDRectangle;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;

@RestController
@RequestMapping("/api/jtdr")
public class JtdrIntegrationController {

    @Autowired
    private JtdrIntegrationService jtdrService;

    @PostMapping("/submit")
    public Map<String, Object> submitCase(@RequestParam("cnr") String cnr) {
        var context = org.dspace.app.rest.utils.ContextUtil.obtainCurrentRequestContext();
        return jtdrService.submitCase(context, cnr);
    }

    @GetMapping("/status/{ackId}")
    public Map<String, Object> getStatus(@PathVariable String ackId) {
        return jtdrService.checkStatus(ackId);
    }

    /** CSV version (Accept: text/csv) */
    @GetMapping(value = "/report", produces = "text/csv")
    public void reportCsv(
            @RequestParam("start") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
            @RequestParam("end")   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end,
            HttpServletResponse response
    ) throws IOException {
        LocalDateTime from = start.atStartOfDay();
        LocalDateTime to   = end.atTime(23, 59, 59, 999_000_000);
        List<JtdrDetailedReportRow> rows = jtdrService.getDetailedReport(from, to);

        response.setContentType("text/csv");
        response.setHeader("Content-Disposition", "attachment; filename=detailed-report.csv");

        try (PrintWriter writer = response.getWriter()) {
            writer.println("Sl No,Batch Name,Case Type,Case No,Case Year,Zip Created At,Zip Created By,Upload Date,Upload Status,File Submitted By,Zip Status");
            for (JtdrDetailedReportRow row : rows) {
                writer.print(row.slNumber); writer.print(",");
                writer.print(csvSafe(row.batchName)); writer.print(",");
                writer.print(csvSafe(row.caseType)); writer.print(",");
                writer.print(csvSafe(row.caseNo)); writer.print(",");
                writer.print(csvSafe(row.caseYear)); writer.print(",");
                writer.print(row.zipCreatedAt != null ? row.zipCreatedAt : ""); writer.print(",");
                writer.print(csvSafe(row.createdBy)); writer.print(",");
                writer.print(row.uploadDate != null ? row.uploadDate : ""); writer.print(",");
                writer.print(csvSafe(row.uploadStatus)); writer.print(",");
                writer.print(csvSafe(row.uploadedBy)); writer.print(",");
                writer.print(csvSafe(row.zipStatus));
                writer.println();
            }
        }
    }

    /** PDF version (Accept: application/pdf) */
    @GetMapping(value = "/report", produces = "application/pdf")
    public void reportPdf(
            @RequestParam("start") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
            @RequestParam("end")   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end,
            HttpServletResponse response
    ) throws IOException {
        LocalDateTime from = start.atStartOfDay();
        LocalDateTime to   = end.atTime(23, 59, 59, 999_000_000);
        List<JtdrDetailedReportRow> rows = jtdrService.getDetailedReport(from, to);

        response.setContentType("application/pdf");
        response.setHeader("Content-Disposition", "attachment; filename=detailed-report.pdf");

        try (PDDocument doc = new PDDocument()) {
            PDPage page = new PDPage(PDRectangle.A4);
            doc.addPage(page);

            float margin = 36; // 0.5 inch
            float y = page.getMediaBox().getHeight() - margin;
            float lineHeight = 14f;
            float left = margin;

            PDPageContentStream cs = new PDPageContentStream(doc, page);
            cs.setFont(PDType1Font.HELVETICA_BOLD, 12);
            cs.beginText();
            cs.newLineAtOffset(left, y);
            cs.showText("JTDR Detailed Report");
            cs.endText();

            y -= (lineHeight * 2);

            // Header row
            String[] headers = new String[] {
                    "Sl No","Batch Name","Case Type","Case No","Upload Date",
                    "Upload Status","Zip Created At","Zip Created By","File Submitted By","Zip Status"
            };

            cs.setFont(PDType1Font.HELVETICA_BOLD, 9);
            y = drawWrappedLine(cs, page, left, y, lineHeight, join(headers, " | "));

            cs.setFont(PDType1Font.HELVETICA, 9);
            for (JtdrDetailedReportRow r : rows) {
                String line = String.format(
                        "%s | %s | %s | %s | %s | %s | %s | %s | %s | %s",
                        safe(r.slNumber),
                        safe(r.batchName),
                        safe(r.caseType),
                        safe(r.caseNo),
                        safe(r.uploadDate),
                        safe(r.uploadStatus),
                        safe(r.zipCreatedAt),
                        safe(r.createdBy),
                        safe(r.uploadedBy),
                        safe(r.zipStatus)
                );
                y = drawWrappedLine(cs, page, left, y, lineHeight, line);
                if (y < margin + lineHeight) { // new page
                    cs.close();
                    page = new PDPage(PDRectangle.A4);
                    doc.addPage(page);
                    cs = new PDPageContentStream(doc, page);
                    cs.setFont(PDType1Font.HELVETICA, 9);
                    y = page.getMediaBox().getHeight() - margin;
                }
            }
            cs.close();

            doc.save(response.getOutputStream());
        }
    }

    private static String csvSafe(Object v) {
        if (v == null) return "";
        String s = v.toString();
        if (s.contains(",") || s.contains("\"")) {
            return "\"" + s.replace("\"", "\"\"") + "\"";
        }
        return s;
    }

    private static String safe(Object v) {
        return v == null ? "" : v.toString();
    }

    private static String join(String[] arr, String sep) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < arr.length; i++) {
            if (i > 0) sb.append(sep);
            sb.append(arr[i]);
        }
        return sb.toString();
    }

    /** Simple line writer with wrapping (approximate) */
    private static float drawWrappedLine(PDPageContentStream cs, PDPage page, float x, float y, float lh, String text) throws IOException {
        float maxWidth = page.getMediaBox().getWidth() - (x * 2);
        String[] words = text.split("\\s+");
        StringBuilder line = new StringBuilder();
        for (String w : words) {
            String trial = line.length() == 0 ? w : line + " " + w;
            float width = PDType1Font.HELVETICA.getStringWidth(trial) / 1000 * 9; // 9pt font
            if (width > maxWidth) {
                cs.beginText(); cs.newLineAtOffset(x, y); cs.showText(line.toString()); cs.endText();
                y -= lh;
                line = new StringBuilder(w);
            } else {
                line = new StringBuilder(trial);
            }
        }
        if (line.length() > 0) {
            cs.beginText(); cs.newLineAtOffset(x, y); cs.showText(line.toString()); cs.endText();
            y -= lh;
        }
        return y;
    }
}












package org.dspace.app.rest.diracai.service;


import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.core.Context;
import org.springframework.web.multipart.MultipartFile;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;

public interface JtdrIntegrationService {
    Map<String, Object> submitCase(Context context, String cnr);
    Map<String, Object> checkStatus(String ackId);
    List<JtdrDetailedReportRow> getDetailedReport(LocalDateTime from, LocalDateTime to);
}


//package org.dspace.app.rest.diracai.service.impl;
//
//import com.fasterxml.jackson.databind.ObjectMapper;
//import lombok.extern.slf4j.Slf4j;
//import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
//import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
//import org.dspace.content.Diracai.FileHashRecord;
//import org.springframework.beans.factory.annotation.Autowired;
//import org.springframework.core.io.FileSystemResource;
//import org.springframework.http.*;
//import org.springframework.stereotype.Service;
//import org.springframework.util.LinkedMultiValueMap;
//import org.springframework.util.MultiValueMap;
//import org.springframework.web.client.RestTemplate;
//import java.net.InetSocketAddress;
//import java.net.Proxy;
//
//import org.springframework.http.ResponseEntity;
//import org.springframework.http.HttpEntity;
//import org.springframework.web.client.RestTemplate;
//import org.springframework.http.HttpHeaders;
//import org.springframework.http.HttpMethod;
//import org.springframework.http.MediaType;
//import org.springframework.http.client.SimpleClientHttpRequestFactory;
//
//import java.io.File;
//import java.util.Map;
//
//@Service
//@Slf4j
//public class JtdrIntegrationServiceImpl implements JtdrIntegrationService {
//
//
//    @Autowired
//    private FileHashRecordRepository repository;
//
//    @Override
//    public Map<String, Object> submitCase(String cnr, String zipHash) {
//        try {
//
//            String folderBasePath = "/home/dspace/dspace/jtdr/";
//            String zipFilePath = folderBasePath + cnr;
//
//            File zipFile = new File(zipFilePath);
//            if (!zipFile.exists()) {
//                return Map.of("error", "ZIP file not found at path", "path", zipFilePath);
//            }
//
//            String url = "https://orissa.jtdr.gov.in/api/add/case";
//
//            HttpHeaders headers = new HttpHeaders();
//            headers.setContentType(MediaType.MULTIPART_FORM_DATA);
//
//            MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
//            body.add("cnr", cnr);
//            body.add("zipHash", zipHash);
//            body.add("caseZip", new FileSystemResource(zipFile));
//            body.add("userId","depositor_hc@orissa.hc.in");
//
//            HttpEntity<MultiValueMap<String, Object>> request = new HttpEntity<>(body, headers);
//// Step 1: Create a request factory with proxy
//            SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
//            Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("127.0.0.1", 8080)); // replace with actual VPN proxy IP & port
//            requestFactory.setProxy(proxy);
//
//// Step 2: Create RestTemplate with proxy
//            RestTemplate restTemplate = new RestTemplate(requestFactory);
//
//// Step 3: Use the same restTemplate for API call
//            ResponseEntity<String> response = restTemplate.postForEntity(apiUrl, requestEntity, String.class);
//
//
//            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);
//
//            // Update FileHashRecord if ackId is present
//            if (responseMap.containsKey("ackId")) {
//                FileHashRecord record = repository.findByFileName(cnr);
//                if (record != null) {
//                    if (responseMap.containsKey("ackId")) {
//                        record.setAckId((String) responseMap.get("ackId"));
//                    }
//                    if (responseMap.containsKey("message")) {
//                        record.setPostResponse((String) responseMap.get("message"));
//                    }
//                    repository.save(record);
//                }
//
//            }
//
//            return responseMap;
//
//        } catch (Exception e) {
//            return Map.of("error", "Failed to submit case", "details", e.getMessage());
//        }
//    }
//
//
//    @Override
//    public Map<String, Object> checkStatus(String ackId) {
//        RestTemplate restTemplate = new RestTemplate();
//        try {
//            String url = "https://orissa.jtdr.gov.in/api/status/case?ackId=" + ackId;
//            ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
//
//            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);
//
//            // Set the 'message' in postResponse
//            if (responseMap.containsKey("message")) {
//                FileHashRecord record = repository.findByAckId(ackId);
//                if (record != null) {
//                    record.setGetCheckResponse((String) responseMap.get("message"));
//                    repository.save(record);
//                }
//            }
//
//            return responseMap;
//        } catch (Exception e) {
//            return Map.of("error", "Failed to get check response", "details", e.getMessage());
//        }
//    }
//
//}




package org.dspace.app.rest.diracai.service.impl;

import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.dspace.xoai.services.api.context.ContextService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.FileSystemResource;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.ssl.SSLContextBuilder;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

import javax.net.ssl.SSLContext;

import java.security.MessageDigest;
import java.nio.file.Files;
import java.nio.file.Path;
import java.math.BigInteger;



import java.io.File;

import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

import static org.dspace.app.rest.diracai.util.InsecureRestTemplateFactory.getInsecureRestTemplate;


@Service
@Slf4j
public class JtdrIntegrationServiceImpl implements JtdrIntegrationService {

    @Autowired
    private FileHashRecordRepository repository;

//    @Override
//    public Map<String, Object> submitCase(String cnr, String zipHash) {
//        try {
//            String folderBasePath = "/home/dspace/dspace/jtdr/";
//            String zipFilePath = folderBasePath + cnr ;
//            File zipFile = new File(zipFilePath);
//
//            if (!zipFile.exists()) {
//                return Map.of("error", "ZIP file not found", "path", zipFilePath);
//            }
//
//            String url = "https://orissa.jtdr.gov.in/api/add/case";
//
//            HttpHeaders headers = new HttpHeaders();
//            headers.setContentType(MediaType.MULTIPART_FORM_DATA);
//
//            String number = cnr.replace(".zip","");
//            MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
//            body.add("cnr", number);
//            body.add("zipHash", zipHash);
//            body.add("caseZip", new FileSystemResource(zipFile));
//            body.add("userId", "depositor_hc@orissa.hc.in");
//
//            HttpEntity<MultiValueMap<String, Object>> request = new HttpEntity<>(body, headers);
//
//            // 🔒 Use the insecure RestTemplate (like curl --insecure)
//            RestTemplate restTemplate = getInsecureRestTemplate();
//
//            ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
//            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);
//
//            if (responseMap.containsKey("ackId")) {
//                FileHashRecord record = repository.findByFileName(cnr);
//                if (record != null) {
//                    record.setAckId((String) responseMap.get("ackId"));
//                    record.setPostResponse((String) responseMap.getOrDefault("message", ""));
//                    repository.save(record);
//                }
//            }
//
//            return responseMap;
//
//        } catch (Exception e) {
//            FileHashRecord record = repository.findByFileName(cnr);
//            if (record != null) {
//                record.setPostResponse(e.getMessage());
//                repository.save(record);
//            }
//            return Map.of("error", "Failed to submit case", "details", e.getMessage());
//        }
//    }

    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private FileHashRecordRepository recordRepository;

    @Autowired private ItemService itemService;
    @Autowired private ContextService contextService;

    private Map<String, String> responseMessages = new HashMap<>();

    @Override
    public List<JtdrDetailedReportRow> getDetailedReport(LocalDateTime from, LocalDateTime to) {
        var records = repository.findAllForReport(from, to);

        try (Context context = contextService.getContext()) {
            // Capture current user name before stream
            String userName = context.getCurrentUser() != null ? context.getCurrentUser().getName() : "Unknown";

            // Use AtomicInteger for counter inside stream
            AtomicInteger counter = new AtomicInteger(1);

            return records.stream().map(rec -> {
                JtdrDetailedReportRow row = new JtdrDetailedReportRow();
                row.batchName = rec.getBatchName();
                row.caseType = rec.getCaseType();
                row.caseNo = rec.getCaseNo();
                row.slNumber = counter.getAndIncrement();
                row.uploadDate = rec.getUploadDate();
                row.userName = userName;
                row.uploadStatus = rec.getGetCheckResponse();
                row.zipCreatedAt = rec.getCreatedAt();
                row.zipStatus = rec.getZipStatus();
                row.createdBy = rec.getCreatedBy();
                row.uploadedBy = rec.getUploadedBy();
                return row;
            }).collect(Collectors.toList());

        } catch (Exception e) {
            return List.of();
        }
    }


    public String Mappers(String status) {
        // Initialize the response messages map
        responseMessages.put("200", "Case Received Successfully");
        responseMessages.put("401", "CNR must not be null or empty.");
        responseMessages.put("402", "Zip hash must not be null or empty.");
        responseMessages.put("403", "Invalid Zip File");
        responseMessages.put("404", "Invalid Zip: Zip name does not match with CNR");
        responseMessages.put("405", "Provided ZIP hash does not match actual file hash.");
        responseMessages.put("406", "Invalid userId");
        responseMessages.put("407", "Provided userId does not match any user in JTDR application");
        responseMessages.put("409", "Duplicate case detected. Case already exists for provided CNR.");
        responseMessages.put("500", "Internal Server Error");

        return responseMessages.get(status);
    }

    @Override
    public Map<String, Object> submitCase(Context context,String cnr) {
        try {

            String dspaceDir = configurationService.getProperty("dspace.dir");
            String folderBasePath = dspaceDir + "/jtdr";
            FileHashRecord fileHashRecord = recordRepository.findByFileName(cnr);
            String cino = fileHashRecord.getCinoNumber();
            String zipFilePath = folderBasePath + "/" + cino + ".zip";
            File zipFile = new File(zipFilePath);


            generateZip(cino, zipFile);

            if (!zipFile.exists()) {
                return Map.of("error", "ZIP file not found", "path", zipFilePath);
            }

            String url = "https://orissa.jtdr.gov.in/api/add/case";

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.MULTIPART_FORM_DATA);


            MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
            String calculatedZipHash = calculateSHA256(zipFile);
            body.add("zipHash", calculatedZipHash);
            body.add("cnr", cino);
            body.add("caseZip", new FileSystemResource(zipFile));
            body.add("userId", "depositor_hc@orissa.hc.in");

            HttpEntity<MultiValueMap<String, Object>> request = new HttpEntity<>(body, headers);
            RestTemplate restTemplate = getInsecureRestTemplate();

            ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);

            String responseText = response.getBody();

            // Extract status code and message from the response text
            String statusCode = extractStatusCode(responseText);


            if (responseMap.containsKey("ackId")) {
                FileHashRecord record = repository.findByFileName(cnr);
                if (record != null) {
                    record.setUploadDate(LocalDateTime.now());
                    record.setAckId((String) responseMap.get("ackId"));
                    record.setPostResponse((String) responseMap.getOrDefault("message", ""));
                    record.setStatus("Transferred Case Successfully");
                    record.setUploadedBy(context.getCurrentUser().getName());
                    repository.save(record);
                }
            } else {
                FileHashRecord record = repository.findByFileName(cnr);
                if (record != null) {
                    record.setUploadDate(LocalDateTime.now());
                    record.setPostStatus(statusCode);
                    record.setPostResponse(Mappers(statusCode));
                    repository.save(record);
                }
            }


            return responseMap;

        } catch (Exception e) {
            FileHashRecord record = repository.findByFileName(cnr);
            if (record != null) {
                record.setPostResponse(e.getMessage());
                repository.save(record);
            }
            return Map.of("error", "Failed to submit case", "details", e.getMessage());
        }
    }

    // Extracts the status code from the response text (e.g., "409")
    private String extractStatusCode(String responseText) {
        String[] parts = responseText.split(" ");
        return parts[0];
    }


    @Override
    public Map<String, Object> checkStatus(String ackId) {
        try {
            String url = "https://orissa.jtdr.gov.in/api/status/case?ackId=" + ackId;

            RestTemplate restTemplate = getInsecureRestTemplate();
            ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);

            if (responseMap.containsKey("message")) {
                FileHashRecord record = repository.findByAckId(ackId);
                if (record != null) {
                    record.setGetCheckResponse((String) responseMap.get("message"));
                    repository.save(record);
                }
            }

            return responseMap;

        } catch (Exception e) {
            return Map.of("error", "Failed to get check response", "details", e.getMessage());
        }
    }



    private void generateZip(String cnr, File zipFile) throws IOException {
        String cleanCnr = cnr.replaceAll("\\.zip$", ""); // Remove .zip if present

        File folderToZip = new File(zipFile.getParent(), cleanCnr); // e.g., /home/dspace/dspace/jtdr/ODHC010004612666
        if (!folderToZip.exists() || !folderToZip.isDirectory()) {
            throw new IOException("Folder to zip not found: " + folderToZip.getAbsolutePath());
        }

        try (ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(zipFile))) {
            File[] files = folderToZip.listFiles();
            if (files == null) throw new IOException("Failed to list files in folder: " + folderToZip);

            for (File file : files) {
                String zipEntryName = cleanCnr + "/" + file.getName(); // ✅ Keep consistent folder name inside zip
                zos.putNextEntry(new ZipEntry(zipEntryName));
                Files.copy(file.toPath(), zos);
                zos.closeEntry();
            }
        }
    }

    private String calculateSHA256(File file) throws Exception {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] fileBytes = Files.readAllBytes(file.toPath());
        byte[] hashBytes = digest.digest(fileBytes);
        BigInteger number = new BigInteger(1, hashBytes);
        StringBuilder hexString = new StringBuilder(number.toString(16));

        while (hexString.length() < 64) {
            hexString.insert(0, '0');
        }
        return hexString.toString();
    }


}





package org.dspace.app.rest.diracai.dto;


import java.time.LocalDateTime;


public class JtdrDetailedReportRow {
    public Integer slNumber;
    public String batchName;
    public String caseType;
    public String caseNo;
    public String zipStatus;
    public LocalDateTime zipCreatedAt;
    public String zipCreatedBy;
    public LocalDateTime uploadDate;
    public String uploadStatus;
    public String userName;
    public String createdBy;
    public String uploadedBy;
    public String caseYear;


}


package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.service.ZipExportService;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.content.Item;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.File;
import java.util.UUID;

@RestController
@RequestMapping("/api/export")
public class ZipExportRestController {

    @Autowired
    private ZipExportService zipExportService;

    @Autowired
    private ItemService itemService;

    @PostMapping("/zip/{itemUUID}")
    public ResponseEntity<String> generateZip(@PathVariable UUID itemUUID) {
        Context context = null;
        try {
            context = ContextUtil.obtainCurrentRequestContext();
            Item item = itemService.find(context, itemUUID);
            if (item == null) {
                return ResponseEntity.status(HttpStatus.NOT_FOUND).body("Item not found");
            }

            File zip = zipExportService.generateZipForItem(context, item);
            return ResponseEntity.ok("Zip generated: " + zip.getAbsolutePath());

        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error: " + e.getMessage());
        } finally {
            if (context != null) {
                context.abort();
            }
        }
    }
}















package org.dspace.app.rest.diracai.service;

import com.amazonaws.util.IOUtils;
import com.fasterxml.jackson.databind.ObjectMapper;
import in.cdac.hcdc.jtdr.metadata.JTDRMetadataSchema;
import in.cdac.hcdc.jtdr.metadata.schema.*;
import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.content.Bitstream;
import org.dspace.content.Bundle;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.content.Item;
import org.dspace.content.MetadataValue;
import org.dspace.content.service.BitstreamService;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.dspace.storage.bitstore.service.BitstreamStorageService;
import java.time.LocalDateTime;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

@Service
public class ZipExportService {

    @Autowired
    private ItemService itemService;

    @Autowired
    private BitstreamService bitstreamService;

    @Autowired
    private BitstreamStorageService bitstreamStorageService;

    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private FileHashRecordRepository fileHashRecordRepository;

    public File generateZipForItem(Context context, Item item) throws Exception {
        String cino = getCinoFromMetadata(item,"dc","cino",null);
        String caseType = getCinoFromMetadata(item,"dc","casetype",null);
        String caseNumber = getCinoFromMetadata(item,"dc","case","number");
        String petitionerName = getCinoFromMetadata(item,"dc","pname",null);
        String respondentName = getCinoFromMetadata(item,"dc","rname",null);
        String advocateName = getCinoFromMetadata(item,"dc","paname",null);
        String judgeName = getCinoFromMetadata(item,"dc","contributor","author");
        String disposalDate = getCinoFromMetadata(item,"dc","date","disposal");
        String district = getCinoFromMetadata(item,"dc","district",null);
        String caseYear = getCinoFromMetadata(item,"dc","caseyear",null);
        String scanDate = getCinoFromMetadata(item,"dc","date","scan");
        String verifiedBy = getCinoFromMetadata(item,"dc","verified-by",null);
        String dateVerification = getCinoFromMetadata(item,"dc","date","verification");
        String batchNumber = getCinoFromMetadata(item,"dc","batch-number",null);
        String barcodeNumber = getCinoFromMetadata(item,"dc","barcode",null);
        String fileSize = getCinoFromMetadata(item,"dc","size",null);
        String characterCount = getCinoFromMetadata(item,"dc","char-count",null);
        String noOfPages = getCinoFromMetadata(item,"dc","pages",null);
        String title = getCinoFromMetadata(item,"dc","title",null);
        String docType = getCinoFromMetadata(item,"dc","doc","type");


        if (cino == null) throw new Exception("CINO not found in metadata");

        String dspaceDir = configurationService.getProperty("dspace.dir");

        File baseDir = new File(dspaceDir, "jtdr");
        if (!baseDir.exists()) baseDir.mkdirs();

        File cinoDir = new File(baseDir, cino);
        if (!cinoDir.exists()) cinoDir.mkdirs();

        List<Map<String, String>> docList = new ArrayList<>();
        for (Bundle bundle : item.getBundles("ORIGINAL")) {
            for (Bitstream bitstream : bundle.getBitstreams()) {
                InputStream is = bitstreamService.retrieve(context, bitstream);
                File file = new File(cinoDir, bitstream.getName());
                try (FileOutputStream fos = new FileOutputStream(file)) {
                    IOUtils.copy(is, fos);
                }
                docList.add(Map.of(
                        "docName", bitstream.getName(),
                        "docType", docType,
                        "docDate", LocalDate.now().format(DateTimeFormatter.ofPattern("dd-MM-yyyy"))
                ));
            }

        }

        // Step 3: Write <CINO>_doc.json
        File jsonFile = new File(cinoDir, cino + "_doc.json");
        new ObjectMapper().writerWithDefaultPrettyPrinter()
                .writeValue(jsonFile, docList);

        // Step 4: Write <CINO>_Metadata.xml
        File xmlFile = new File(cinoDir, cino + "_Metadata.xml");
        generateMetadataXml(item, cino ,caseType, caseNumber,petitionerName,respondentName,advocateName, judgeName ,disposalDate,district, caseYear,scanDate,verifiedBy,dateVerification,barcodeNumber,fileSize,characterCount,noOfPages,title,docType,xmlFile);


        // Step 5: Create zip file
        File zipFile = new File(baseDir, cino + ".zip");
        try (FileOutputStream fos = new FileOutputStream(zipFile);
             ZipOutputStream zos = new ZipOutputStream(fos)) {
            zipFolder(cinoDir, cinoDir.getName(), zos);
        }

        FileHashRecord fileHashRecord = new FileHashRecord();
        fileHashRecord.setFileName(caseType+"_"+title+"_"+caseYear);
        fileHashRecord.setBatchName(batchNumber);
        fileHashRecord.setCaseType(caseType);
        fileHashRecord.setCaseNo(caseNumber);
        fileHashRecord.setStatus("Zip File Created");
        fileHashRecord.setCreatedAt(LocalDateTime.now());
        fileHashRecord.setCinoNumber(cino);
        fileHashRecord.setFileCount(docList.size());
        fileHashRecord.setCreatedBy(context.getCurrentUser().getName());
        fileHashRecordRepository.save(fileHashRecord);
        return zipFile;
    }


    private String getCinoFromMetadata(Item item,String schema, String element , String qualifier) {
        List<MetadataValue> values = itemService.getMetadata(item, schema, element, qualifier, Item.ANY);
        return values.isEmpty() ? null : values.get(0).getValue();
    }

    private void generateMetadataXml(
            Item item,
            String cino,
            String caseType,
            String caseNumber,
            String petitionerName,
            String respondentName,
            String advocateName,
            String judgeName,
            String disposalDate,
            String district,
            String caseYear,
            String scanDate,
            String verifiedBy,
            String dateVerification,
            String barcodeNumber,
            String fileSize,
            String characterCount,
            String noOfPages,
            String title,
            String docType,
            File xmlOutputFile
    )

    {
        ECourtCaseType eCourtCaseType = new ECourtCaseType();

        System.out.println(
                "cino=" + cino +
                        ", caseType=" + caseType +
                        ", caseNumber=" + caseNumber +
                        ", petitionerName=" + petitionerName +
                        ", respondentName=" + respondentName +
                        ", advocateName=" + advocateName +
                        ", judgeName=" + judgeName +
                        ", disposalDate=" + disposalDate +
                        ", district=" + district +
                        ", caseYear=" + caseYear +
                        ", scanDate=" + scanDate +
                        ", verifiedBy=" + verifiedBy +
                        ", dateVerification=" + dateVerification +
                        ", barcodeNumber=" + barcodeNumber +
                        ", fileSize=" + fileSize +
                        ", characterCount=" + characterCount +
                        ", noOfPages=" + noOfPages +
                        ", title=" + title
        );

        CaseTypeInformation caseTypeInformation = new CaseTypeInformation();
        caseTypeInformation.setCaseCNRNumber(cino);
        caseTypeInformation.setCaseNature(caseType);
        caseTypeInformation.setCaseNumber(title);
        caseTypeInformation.setCaseTypeName(caseType);
        caseTypeInformation.setNameOfDistrict(district);
        caseTypeInformation.setRegistrationYear(caseYear);
        caseTypeInformation.setRegistrationDate(caseYear+"-01-01 00:00:00");
        caseTypeInformation.setRegistrationNumber(title);
//        caseTypeInformation.setDocType(docType);

        eCourtCaseType.setCase(caseTypeInformation);

        LitigantTypeInformation litigant = new LitigantTypeInformation();
        PetitionerTypeInformation petitioner = new PetitionerTypeInformation();
        petitioner.setPetitionerName(petitionerName);
        litigant.getPetitioner().add(petitioner);


        RespondentTypeInformation respondent = new RespondentTypeInformation();
        respondent.setRespondentName(respondentName);
        litigant.getRespondent().add(respondent);

        caseTypeInformation.setLitigant(litigant);

        JudgeTypeInformation judgeTypeInformation = new JudgeTypeInformation();

        JudgeInformation judgeInformation = new JudgeInformation();
        judgeInformation.setJudgeName(judgeName);
        judgeTypeInformation.getJudgeInfo().add(judgeInformation);
        eCourtCaseType.setJudge(judgeTypeInformation);


        StatusOfCasesTypeInformation statusOfCasesTypeInformation = new StatusOfCasesTypeInformation();
        statusOfCasesTypeInformation.setDateOfDisposal(disposalDate);
        eCourtCaseType.setStatusOfCases(statusOfCasesTypeInformation);


        DigitizationTypeInformation digitizationTypeInformation = new DigitizationTypeInformation();
        MasterFileType masterFileType = new MasterFileType();
        masterFileType.setFileSize(fileSize);
        digitizationTypeInformation.setMasterFile(masterFileType);
        digitizationTypeInformation.setVerifiedBy(verifiedBy);
        eCourtCaseType.setDigitization(digitizationTypeInformation);





        AdvocateTypeInformation advocate = new AdvocateTypeInformation();
        advocate.setAdvocateName(advocateName);
        caseTypeInformation.getAdvocate().add(advocate);


        JTDRMetadataSchema.createXML(eCourtCaseType, xmlOutputFile.getAbsolutePath());
    }



    private void zipFolder(File folder, String basePath, ZipOutputStream zos) throws Exception {
        for (File file : folder.listFiles()) {
            if (file.isDirectory()) {
                zipFolder(file, basePath + "/" + file.getName(), zos);
            } else {
                try (FileInputStream fis = new FileInputStream(file)) {
                    ZipEntry entry = new ZipEntry(basePath + "/" + file.getName());
                    zos.putNextEntry(entry);
                    IOUtils.copy(fis, zos);
                    zos.closeEntry();
                }
            }
        }
    }
}






package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "file_hash_record")
public class FileHashRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String fileName;
    private String hashValue;
    private LocalDateTime createdAt;
    private String ackId;
    private String zipStatus;
    private String postResponse;
    private String postStatus;
    private String getCheckResponse;
    private String getCheckStatus;
    private Integer fileCount;
    private LocalDateTime uploadDate;
    private String batchName;
    private String caseType;
    private String caseNo;
    private String Status;
    private String cinoNumber;
    private String createdBy;
    private String uploadedBy;

    public String getCreatedBy() {
        return createdBy;
    }

    public String getUploadedBy() {
        return uploadedBy;
    }

    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }

    public void setUploadedBy(String uploadedBy) {
        this.uploadedBy = uploadedBy;
    }

    public String getCinoNumber() {
        return cinoNumber;
    }

    public void setCinoNumber(String cinoNumber) {
        this.cinoNumber = cinoNumber;
    }

    public String getStatus() {
        return Status;
    }

    public void setStatus(String status) {
        Status = status;
    }

    public void setUploadDate(LocalDateTime uploadDate) {
        this.uploadDate = uploadDate;
    }

    public void setFileCount(Integer fileCount) {
        this.fileCount = fileCount;
    }

    public LocalDateTime getUploadDate() {
        return uploadDate;
    }

    public void setCaseType(String caseType) {
        this.caseType = caseType;
    }

    public void setBatchName(String batchName) {
        this.batchName = batchName;
    }

    public String getCaseType() {
        return caseType;
    }

    public String getCaseNo() {
        return caseNo;
    }

    public String getBatchName() {
        return batchName;
    }

    public void setCaseNo(String caseNo) {
        this.caseNo = caseNo;
    }

    public Integer getFileCount() {
        return fileCount;
    }

    public void setFileCount(int fileCount) {
        this.fileCount = fileCount;
    }

    @PrePersist
    public void onCreate() {
        this.createdAt =  LocalDateTime.now();
    }


    public String getZipStatus() {
        return zipStatus;
    }

    public String getGetCheckResponse() {
        return getCheckResponse;
    }

    public String getGetCheckStatus() {
        return getCheckStatus;
    }

    public String getPostResponse() {
        return postResponse;
    }

    public String getPostStatus() {
        return postStatus;
    }

    public void setGetCheckResponse(String getCheckResponse) {
        this.getCheckResponse = getCheckResponse;
    }

    public void setGetCheckStatus(String getCheckStatus) {
        this.getCheckStatus = getCheckStatus;
    }

    public void setPostResponse(String postResponse) {
        this.postResponse = postResponse;
    }

    public void setPostStatus(String postStatus) {
        this.postStatus = postStatus;
    }

    public void setZipStatus(String zipStatus) {
        this.zipStatus = zipStatus;
    }

    public FileHashRecord() {}


    // Getters and setters

    public void setAckId(String ackId) {
        this.ackId = ackId;
    }

    public String getAckId() {
        return ackId;
    }

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getFileName() { return fileName; }

    public void setFileName(String fileName) { this.fileName = fileName; }

    public String getHashValue() { return hashValue; }

    public void setHashValue(String hashValue) { this.hashValue = hashValue; }

    public LocalDateTime getCreatedAt() { return createdAt; }

    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}

 -->