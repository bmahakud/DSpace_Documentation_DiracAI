
# Bulk Upload Feature - Backend Documentation

## Overview
This module manages **bulk upload requests** with functionalities for:
- Uploading ZIP files containing items.
- Extracting and storing metadata (`dublin_core.xml`).
- Approving or rejecting uploaded batches by Admin + Reviewer users.
- Tracking status (CLAIMED, APPROVED, REJECTED).
- Viewing pooled tasks.

---

## API Endpoints (`/api/bulk-upload`)

### 1. **Upload Bulk File**
**POST** `/upload/{uuid}`  
Uploads a ZIP file for a given collection UUID.  
- **Request Param:** `file` (Multipart file)
- **Path Variable:** `uuid` (Collection UUID)
- **Response:** `BulkUploadRequest` object.

---

### 2. **Approve Bulk Upload**
**POST** `/approve/{uuid}`  
Approves a bulk upload request. Only accessible to users who are **both Admin and Reviewer**.  
- **Path Variable:** `uuid` (Request UUID)
- **Response:** Updated `BulkUploadRequest`.

---

### 3. **Reject Bulk Upload**
**POST** `/reject/{uuid}`  
Rejects a bulk upload request. Requires **Admin + Reviewer** role.  
- **Path Variable:** `uuid`
- **Response:** Updated `BulkUploadRequest`.

---

### 4. **Fetch All Upload Requests**
**GET** `/all`  
Retrieves all bulk upload requests.  
- **Response:** List of `BulkUploadRequest`.

---

### 5. **Get Upload Requests by Status**
**GET** `/status/{status}`  
Fetches all bulk requests filtered by status (`CLAIMED`, `APPROVED`, `REJECTED`).  
- **Path Variable:** `status`
- **Response:** List of `BulkFileDto`.

---

### 6. **Fetch Pooled Tasks**
**GET** `/pooled`  
Returns tasks available for the logged-in reviewer.  
- If Admin → returns all except CLAIMED.  
- If Reviewer → returns requests they are assigned or uploaded.  
- **Response:** List of `BulkFileDto`.

---

### 7. **Get Batch Files**
**GET** `/{uuid}`  
Fetches details of a batch and associated items with metadata.  
- **Response:** `BulkUploadRequestResponseDTO` (includes files + parsed metadata).

---

## Services

### **BulkUploadRequestService**
- `createRequest(Context, MultipartFile, UUID)` → Creates new bulk upload request.
- `approveRequest(UUID, Context, AuthTokenPayload)` → Approves a batch.
- `rejectRequest(UUID, Context)` → Rejects a batch.
- `findAll()` → Returns all requests.
- `findByStatus(Context, String)` → Returns requests by status.
- `getFile(UUID)` → Fetch details of request + items + metadata.
- `getPooledTasksForReviewer(Context, UUID)` → Returns reviewer-specific tasks.

### **BulkUploadItemService**
- `create(Context, BulkUploadItem)` → Saves new upload item.

---

## Repositories

### **BulkUploadRequestRepository**
- `findByStatus(String)` → Fetch requests by status.
- `findAllByStatusNot(String)` → Fetch all excluding a status.
- `findByReviewerId(UUID)` → Fetch requests by reviewer.
- `findByUploaderId(UUID)` → Fetch requests by uploader.

### **BulkUploadItemRepository**
- `findWithMetadataByUploadRequest(UUID)` → Fetch items with metadata for given upload request.

---

## Utilities

### **BulkUploadRequestUtil**
- `handleZipUpload(Context, MultipartFile, UUID)` → Handles saving, extracting ZIP, creating `BulkUploadItem` entries, and parsing metadata.
- `parseDublinCoreXML(File, BulkUploadItem)` → Extracts Dublin Core XML metadata and persists it.

---

## Entity Relationships
- **BulkUploadRequest** → Represents a batch upload request (status, uploader, reviewer, filename, collection).
- **BulkUploadItem** → Represents an item inside the batch (linked to request).
- **BulkUploadItemMetadata** → Metadata key-value pairs extracted from `dublin_core.xml`.

---

## Status Flow
1. **CLAIMED** → When request is created.
2. **APPROVED** → After admin + reviewer approval.
3. **REJECTED** → After admin + reviewer rejection.

---

## Security Rules
- **Upload** → Requires user to be **Admin + Uploader**.
- **Approve/Reject** → Requires user to be **Admin + Reviewer**.

---

## File Storage
- Uploaded ZIP is stored at:  
  ```
  {dspace.dir}/bulk_upload/{requestUUID}/{requestUUID}.zip
  ```
- Contents are extracted, and metadata (`dublin_core.xml`) is parsed.

---
