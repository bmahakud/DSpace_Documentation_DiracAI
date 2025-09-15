
# Backend API Documentation — `backend.md`

> Detailed documentation for the DSpace JTDR & FileHash backend code.  
> Place this file in your repository (suggested: `/backend.md` or `/docs/backend.md`) and commit.

---

## Table of contents

1. [Overview](#overview)  
2. [Repository / file layout (created or modified)](#repository--file-layout-created-or-modified)  
3. [Common patterns & utilities referenced](#common-patterns--utilities-referenced)  
4. [Entities and DTOs](#entities-and-dtos)  
5. [Repositories](#repositories)  
6. [Controllers (endpoints) — detailed explanation](#controllers-endpoints---detailed-explanation)  
   - [FileHashController (`/api/cnr`)](#filehashcontroller-apicnr)  
   - [JtdrIntegrationController (`/api/jtdr`)](#jtdrintegrationcontroller-apijtdr)  
   - [ZipExportRestController (`/api/export`)](#zipexportrestcontroller-apiexport)  
7. [Services (interfaces + impl)](#services-interfaces--impl)  
   - [JtdrIntegrationServiceImpl](#jtdrintegrationserviceimpl)  
   - [ZipExportService](#zipexportservice)  
8. [Helper logic and algorithms](#helper-logic-and-algorithms)  
9. [Sample requests (curl)](#sample-requests-curl)  
10. [Where to put this file in GitHub & example git commands](#where-to-put-this-file-in-github--example-git-commands)  
11. [Important notes, potential bugs, and suggested improvements](#important-notes-potential-bugs-and-suggested-improvements)  

---

## Overview

This backend integrates a DSpace instance with an external JTDR service. It includes:

- generation of a ZIP package per case (CINO),
- writing `<CINO>_doc.json` and `<CINO>_Metadata.xml`,
- storing metadata in a `file_hash_record` database table (entity `FileHashRecord`),
- computing SHA-256 for ZIPs and optionally submitting them to JTDR,
- endpoints to list/filter/delete zip records and to submit/check status with JTDR,
- CSV and PDF reporting over a date range.

The documentation below explains each controller, service and repository in detail and points where new files were created / changed.

---

## Repository / file layout (created or modified)

Suggested package root: `src/main/java/org/dspace/app/rest/diracai/...`

Files you provided / modified:

- `controller/`
  - `FileHashController.java` — API for generating zip/hash, listing and deleting records.
  - `JtdrIntegrationController.java` — API for submitting case, checking status, downloading report (CSV/PDF).
  - `ZipExportRestController.java` — API to generate zip for a DSpace `Item` by its UUID.
- `Repository/`
  - `FileHashRecordRepository.java` — JPA repository with custom queries.
- `service/`
  - `JtdrIntegrationService.java` — interface
  - `service/impl/JtdrIntegrationServiceImpl.java` — implementation containing JTDR calls, zip generation, SHA-256.
  - `ZipExportService.java` — service that extracts bitstreams, writes JSON + XML, zips.
- `dto/`
  - `JtdrDetailedReportRow.java` — DTO used in reports.
- `content/Diracai/`
  - `FileHashRecord.java` — JPA entity (table: `file_hash_record`)

Utilities referenced (verify presence):
- `org.dspace.app.rest.diracai.util.InsecureRestTemplateFactory` (method `getInsecureRestTemplate()`)
- `org.dspace.app.rest.utils.ContextUtil` (to obtain DSpace Context)
- `in.cdac.hcdc.jtdr.metadata.JTDRMetadataSchema` (external library to generate JTDR XML)

---

## Common patterns & utilities referenced

- **DSpace Context**: Use `Context` for retrieving current user and repository operations. Controllers use `ContextUtil.obtainCurrentRequestContext()` (preferred) rather than expecting Spring to inject `Context` directly as a controller parameter.

- **RestTemplate (insecure)**: `getInsecureRestTemplate()` is used to create a RestTemplate that ignores SSL (useful for testing with self-signed certs). Do not use in production.

- **Filesystem workspace**: Files are stored under `${dspace.dir}/jtdr/<CINO>/` and zipped to `${dspace.dir}/jtdr/<CINO>.zip`.

---

## Entities and DTOs

### `FileHashRecord` (JPA entity — `content/Diracai/FileHashRecord.java`)

Important fields:
- `id` — Primary key (Long).
- `fileName` — Human readable name (used in some lookups).
- `hashValue` — SHA-256 of zip (optional).
- `createdAt` — set by `@PrePersist public void onCreate()` to `LocalDateTime.now()`.
- `ackId` — acknowledgement id returned by JTDR.
- `zipStatus`, `postResponse`, `postStatus`, `getCheckResponse`, `getCheckStatus` — store JTDR interaction details and statuses.
- `fileCount` — number of bitstreams included.
- `uploadDate` — timestamp when the case was transmitted.
- `batchName`, `caseType`, `caseNo`, `cinoNumber`, `createdBy`, `uploadedBy` — metadata.
- **Note**: field `Status` is capitalized (non-idiomatic). Consider renaming to `status`.

---

## Repositories

### `FileHashRecordRepository`

Extends `JpaRepository<FileHashRecord, Long>`. Custom methods:
- `FileHashRecord findByFileName(String cnr)` — exact match on `fileName`.
- `FileHashRecord findByAckId(String ackId)` — lookup by acknowledgement id.
- `List<FileHashRecord> findAll(Sort sort)` — return sorted list.
- `@Query(...) List<FileHashRecord> findAllForReport(LocalDateTime from, LocalDateTime to)` — returns records with `createdAt` between `from` and `to`, ordered descending.
- `Page<FileHashRecord> findByAckIdIsNotNullAndAckIdNot(Pageable pageable, String emptyValue)` — returns submitted records (ackId present and not equal to `emptyValue`).
- `Page<FileHashRecord> findByAckIdIsNullOrAckId(Pageable pageable, String emptyValue)` — returns not-submitted records (ackId null or equal to `emptyValue`).

**Implementation note**: using `""` (empty string) as a sentinel is a hack; prefer to store `null` for missing `ackId` consistently.

---

## Controllers (endpoints) — detailed explanation

> For each controller we give the endpoint path, parameters, purpose, and important logic lines.

---

### FileHashController (`/api/cnr`)

**File:** `controller/FileHashController.java`

**Purpose:** generate ZIPs and hashes, list DB records with paging/filtering, and delete generated zips/DB records.

#### Endpoints

1. `POST /api/cnr/generate?cnr=<cnr>&docType=<docType>`
   - Method: `generateAndStoreHash(@RequestParam("cnr") String cnr, @RequestParam("docType") String docType, Context context)`
   - Behavior:
     - Delegates to `fileHashService.generateZipAndHash(cnr, context, docType)`.
     - Returns the `FileHashRecord` created/saved by the service.
   - Important note:
     - Passing `Context` as a controller method parameter will not be injected by Spring by default. Use:
       ```java
       var context = ContextUtil.obtainCurrentRequestContext();
       ```
       inside the method, or register a HandlerMethodArgumentResolver.

2. `GET /api/cnr/records?page=0&size=10&sortBy=createdAt&sortDir=desc&submitted=submit|notSubmitted`
   - Method: `getAllHashes(...)`
   - Purpose: paging & sorting of `FileHashRecord` table; optional `submitted` filter.
   - Logic:
     - Build `Sort` from `sortBy` and `sortDir`.
       ```java
       Sort sort = sortDir.equalsIgnoreCase("asc")
                     ? Sort.by(sortBy).ascending()
                     : Sort.by(sortBy).descending();
       Pageable pageable = PageRequest.of(page, size, sort);
       ```
     - If `submitted=submit`: call `findByAckIdIsNotNullAndAckIdNot(pageable, "")`.
     - If `submitted=notSubmitted`: call `findByAckIdIsNullOrAckId(pageable, "")`.
     - Else call `findAll(pageable)`.
   - Response: `200 OK` containing `Page<FileHashRecord>`, or `500` on error.

3. `DELETE /api/cnr/records/{fileName}` and alias `DELETE /api/cnr/zip/{fileName}`
   - Method: `deleteGeneratedRecord(@PathVariable String fileName)` / `deleteGeneratedZip(...)`
   - Purpose: delete the generated ZIP file and its DB record.
   - Logic:
     - Calls `fileHashService.deleteZipAndRecord(fileName)` which returns an enum `DeleteResult`.
     - If `NOT_FOUND` => return 404.
     - Otherwise return 204 No Content.
   - Errors: `IOException` => 500

**Where changes were made**: this controller was added/modified in `controller/FileHashController.java`.

---

### JtdrIntegrationController (`/api/jtdr`)

**File:** `controller/JtdrIntegrationController.java`

**Purpose:** Submit cases to JTDR, check status using ackId, and generate reports (CSV and PDF) for a date range.

#### Endpoints

1. `POST /api/jtdr/submit?cnr=<cnr>`
   - Method: `submitCase(@RequestParam("cnr") String cnr)`
   - Behavior:
     - Obtain DSpace context:
       ```java
       var context = org.dspace.app.rest.utils.ContextUtil.obtainCurrentRequestContext();
       ```
     - Delegate to `jtdrService.submitCase(context, cnr)`.
   - Core idea: submit the ZIP and SHA-256 hash for the CINO to JTDR, capture response and update DB record.

2. `GET /api/jtdr/status/{ackId}`
   - Method: `getStatus(@PathVariable String ackId)`
   - Behavior: delegates to `jtdrService.checkStatus(ackId)` and returns the parsed response map.

3. `GET /api/jtdr/report?start=YYYY-MM-DD&end=YYYY-MM-DD` — CSV variant
   - `@GetMapping(value = "/report", produces = "text/csv")`
   - Method: `reportCsv(LocalDate start, LocalDate end, HttpServletResponse response)`
   - Behavior:
     - Convert `start` and `end` to `LocalDateTime` range (start-of-day, end-of-day).
     - Call `jtdrService.getDetailedReport(from, to)`.
     - Stream CSV rows to the response with header:
       ```
       Sl No,Batch Name,Case Type,Case No,Case Year,Zip Created At,Zip Created By,Upload Date,Upload Status,File Submitted By,Zip Status
       ```
     - Use `csvSafe(...)` to escape commas/quotes.

4. `GET /api/jtdr/report?start=...&end=...` — PDF variant
   - `@GetMapping(value = "/report", produces = "application/pdf")`
   - Method: `reportPdf(...)`
   - Behavior:
     - Same date range and `getDetailedReport`.
     - Uses **Apache PDFBox** to produce a PDF with wrapped lines.
     - Key helper functions:
       - `drawWrappedLine(PDPageContentStream cs, PDPage page, float x, float y, float lh, String text)` — approximate text wrapping by measuring width via `PDType1Font.HELVETICA.getStringWidth(trial) / 1000 * fontSize`.
       - `join(String[] arr, String sep)` — helper join for headers.
   - Notes:
     - PDF generation uses an approximate wrapping; for complex tables consider a proper table helper library.

**Where changes were made**: `controller/JtdrIntegrationController.java`.

---

### ZipExportRestController (`/api/export`)

**File:** `controller/ZipExportRestController.java`

**Purpose:** Create a zip for a DSpace `Item` by UUID using `ZipExportService`.

#### Endpoint

`POST /api/export/zip/{itemUUID}`
- Method: `generateZip(@PathVariable UUID itemUUID)`
- Behavior:
  - Obtain DSpace `Context` using `ContextUtil.obtainCurrentRequestContext()`.
  - `Item item = itemService.find(context, itemUUID)`; if null, return 404.
  - Call `zipExportService.generateZipForItem(context, item)`. Return path of created zip.
  - Always `context.abort()` in `finally` block.

**Where changes were made**: `controller/ZipExportRestController.java`.

---

## Services (interfaces + impl)

### JtdrIntegrationServiceImpl (`service/impl/JtdrIntegrationServiceImpl.java`)

**Responsibilities**
- Build (or re-generate) zip for a CINO (via `generateZip()`), compute SHA-256, and submit as `multipart/form-data` to JTDR endpoint `https://orissa.jtdr.gov.in/api/add/case`.
- Parse JTDR response; if `ackId` present, update `FileHashRecord` (`ackId`, `uploadDate`, `postResponse`, `status`, `uploadedBy`) and save to DB. Otherwise, map status codes to messages and store `postStatus` and `postResponse`.
- Provide `checkStatus(ackId)` which calls `https://orissa.jtdr.gov.in/api/status/case?ackId=<ackId>`, parse JSON and write `message` into DB.
- Provide `getDetailedReport(from, to)` — uses `FileHashRecordRepository.findAllForReport(from, to)` then maps to DTO rows, capturing current user name via `ContextService.getContext()`.

**Key helper methods in this class**
- `generateZip(String cnr, File zipFile)`: zips files under the folder `<zipFile.parent>/<cleanCnr>/` into `zipFile`. Each zip entry name is `cleanCnr/<fileName>`.
- `calculateSHA256(File file)`: computes SHA-256 by reading file bytes and digesting via `MessageDigest`. (Note: reads entire file into memory — consider using a `DigestInputStream` for large files.)
- `extractStatusCode(String responseText)`: extracts status code by splitting response text. *Fragile* — prefer parsing JSON or explicit status field.
- `Mappers(String status)`: maps numeric status strings to human readable messages (200, 401, 402, ..., 500).

**Important lines & behavior**
- Determine the zip path from config: `String dspaceDir = configurationService.getProperty("dspace.dir"); String folderBasePath = dspaceDir + "/jtdr";`
- Uses `getInsecureRestTemplate()` to POST `Multipart` request (zip file + zipHash + cnr + userId).
- After calling JTDR endpoint, parse response JSON into a `Map` (`ObjectMapper().readValue(response.getBody(), Map.class)`).
- Update DB record depending on presence/absence of `ackId`.

**Where changes were made**: `service/impl/JtdrIntegrationServiceImpl.java`.

---

### ZipExportService (`service/ZipExportService.java`)

**Responsibilities**
- Read `Item` metadata (dc schema) using `itemService.getMetadata(...)`.
- Copy `ORIGINAL` bitstreams to local directory `${dspace.dir}/jtdr/<CINO>/` using `bitstreamService.retrieve(context, bitstream)` and `IOUtils.copy(is, fos)`.
- Build `docList` (list of maps with docName/docType/docDate) and write `<CINO>_doc.json`.
- Build `<CINO>_Metadata.xml` from metadata by constructing `ECourtCaseType` and calling `JTDRMetadataSchema.createXML(...)`.
- Zip folder into `<CINO>.zip` using `zipFolder(...)`.
- Persist a `FileHashRecord` with fileName (constructed), batchName, caseType, caseNo, status, createdAt, cinoNumber, fileCount, createdBy.

**Important details**
- Use of `item.getBundles("ORIGINAL")` ensures only original bitstreams are exported.
- `generateMetadataXml(...)` builds the JTDR XML using data classes like `CaseTypeInformation`, `LitigantTypeInformation`, `JudgeTypeInformation`, `DigitizationTypeInformation`, etc.

**Where changes were made**: `service/ZipExportService.java`.

---

## Helper logic and algorithms

### ZIP creation
- `ZipOutputStream` is used to create zip files. Entries are created using `ZipEntry` and then file bytes are copied into the stream.
- `zipFolder(File folder, String basePath, ZipOutputStream zos)` recursively walks directories and writes files.

### SHA-256 hashing
- Current method `calculateSHA256(File file)` reads full file into memory via `Files.readAllBytes` and computes digest using `MessageDigest.getInstance("SHA-256")`. For large files replace with streaming digest using `DigestInputStream` to avoid OOM.

### JTDR multipart upload
- The multipart body contains:
  - `cnr` — case identifier (CINO)
  - `zipHash` — SHA-256 hex
  - `caseZip` — actual zip as `FileSystemResource`
  - `userId` — hardcoded `depositor_hc@orissa.hc.in`
- Content type set to `MediaType.MULTIPART_FORM_DATA`

### CSV and PDF report generation
- CSV: uses `csvSafe()` to quote fields with commas or double-quotes and write to `HttpServletResponse` `PrintWriter`.
- PDF: uses Apache PDFBox, primitive wrapping via `drawWrappedLine(...)` measuring string width using `PDType1Font.getStringWidth()`.

---

## Sample requests (curl)

Generate ZIP & DB record (delegates to service):
```bash
curl -X POST "http://<host>/api/cnr/generate?cnr=<CNR_OR_FILE_NAME>&docType=order"
```

List records (paged):
```bash
curl "http://<host>/api/cnr/records?page=0&size=10&sortBy=createdAt&sortDir=desc"
```

Submit case to JTDR:
```bash
curl -X POST "http://<host>/api/jtdr/submit?cnr=<fileName_or_cnr>"
```

Check JTDR status:
```bash
curl "http://<host>/api/jtdr/status/<ackId>"
```

Download CSV report:
```bash
curl -X GET "http://<host>/api/jtdr/report?start=2025-01-01&end=2025-01-31" -H "Accept: text/csv" -o detailed-report.csv
```

Download PDF report:
```bash
curl -X GET "http://<host>/api/jtdr/report?start=2025-01-01&end=2025-01-31" -H "Accept: application/pdf" -o detailed-report.pdf
```

Generate zip for item (by UUID):
```bash
curl -X POST "http://<host>/api/export/zip/<item-uuid>"
```

---

## Where to put this file in GitHub & example git commands

Suggested location: repository root as `backend.md` or under `/docs/backend.md`.

Example (local repo):
```bash
# from repo root
cp /path/to/backend.md backend.md
git checkout -b docs/add-backend-doc
git add backend.md
git commit -m "docs: add backend API documentation (JTDR + FileHash)"
git push origin docs/add-backend-doc
# Then open a PR on GitHub to merge
```

If you want me to generate a shorter `README`-style file instead for your repo root, I can produce that as well.

---

## Important notes, potential bugs, and suggested improvements

1. **Context argument in controllers**  
   - Do not declare `Context` as a controller method parameter unless you have a custom argument resolver. Instead call:
     ```java
     var context = ContextUtil.obtainCurrentRequestContext();
     ```

2. **Consistency for `fileName` vs `cino`**  
   - In some methods, controllers pass `cnr` which is used to find `FileHashRecord` by `fileName`, then `cinoNumber` is used to compute zip filename. Standardize parameter names between UI, API, DB and service layer. Prefer `cino` or `fileName` consistently.

3. **Avoid storing empty string for ackId**  
   - Standardize missing ackId as `NULL` rather than `""`. Remove the `ackIdNotEmpty` workaround once normalized.

4. **Large file handling**  
   - Replace `Files.readAllBytes()` in `calculateSHA256()` with streaming digest using `MessageDigest` + `DigestInputStream`.

5. **Robust JTDR response parsing**  
   - `extractStatusCode(String responseText)` uses split on spaces — fragile. Prefer parsing JSON and extracting a `status` or `code` field, or use HTTP status codes.

6. **Transactionality**  
   - Consider `@Transactional` on service methods that perform DB updates so that partial updates don't leave inconsistent DB state on failure.

7. **Security**  
   - `getInsecureRestTemplate()` should be used only for test environments. For production set up proper TLS certificates and trust stores.

8. **Logging & error handling**  
   - Add `log.debug/info/error` with structured messages; avoid storing raw exception messages into DB fields (sanitize).

9. **Field naming**  
   - Rename `Status` field in `FileHashRecord` to `status` to follow Java Bean naming conventions.

10. **Resource cleanup**  
   - Use try-with-resources for `InputStream` / `OutputStream` across the codebase to avoid resource leaks.

11. **Unit & integration tests**  
   - Add tests for: zip generation, zip SHA-256 verification, JTDR upload (mocking remote server), CSV/PDF generation.

---

## Final notes

- I saved this documentation to a markdown file named `backend.md` for you to download and commit.  
- If you want, I can:
  - Create a shorter `README.md` ready for opening a PR,
  - Produce unit test skeletons (JUnit + Mockito) for `JtdrIntegrationServiceImpl` and `ZipExportService`,
  - Or convert this into HTML or PDF for sharing.

