
---

# 📘 Watermark API Documentation

## Purpose

The **Watermark API** provides a way to upload and retrieve a watermark image in DSpace.

* Administrators can upload a watermark image (e.g., PNG/JPEG).
* The image is stored under `$DSPACE_DIR/watermark/` as `image.<extension>`.
* The API exposes endpoints to **fetch the current watermark** or **upload a new one**.
* Only one watermark can exist at a time (new uploads replace the old one).

This watermark is later applied dynamically during PDF rendering or download workflows.

---

## API Endpoints

### 1. **GET /api/watermark**

Retrieve the current watermark image.

#### Behavior

* Looks in `$DSPACE_DIR/watermark/`.
* Returns the **first file starting with `image.`** (e.g., `image.png`, `image.jpg`).
* If no watermark exists → returns **204 No Content**.
* If folder not found → returns **404 Not Found**.

#### Response

* **200 OK** with watermark binary content.
* Headers:

  * `Content-Disposition: inline; filename="image.png"`
  * `Content-Type: image/png` (or detected MIME type)

#### Example (cURL)

```bash
curl -X GET http://<server>/api/watermark --output watermark.png
```

---

### 2. **POST /api/watermark**

Upload a new watermark file.

#### Authorization

* Restricted to **ADMIN** or **SYSTEM\_ADMIN** roles (`@PreAuthorize`).

#### Request

* Multipart form-data with field `file`.
* Accepted: PNG, JPEG, etc.

#### Behavior

1. Creates `$DSPACE_DIR/watermark/` if missing.
2. Deletes any existing `image.*` file.
3. Saves the new watermark as `image.<original_extension>`.

#### Response

* **201 Created** with JSON body:

```json
{
  "filename": "image.png",
  "contentType": "image/png",
  "message": "Watermark uploaded successfully"
}
```

* **400 Bad Request** → if no file provided.
* **500 Internal Server Error** → on storage failure.

#### Example (cURL)

```bash
curl -X POST http://<server>/api/watermark \
  -H "Authorization: Bearer <token>" \
  -F "file=@/path/to/watermark.png"
```

---

## File Storage Logic

* **Location**:
  All watermark files are stored under:

  ```
  $DSPACE_DIR/watermark/
  ```

  Example: `/dspace/watermark/image.png`

* **Naming Rule**:
  Always saved as `image.<ext>` (preserves original extension).
  Example: If uploaded `logo.jpg` → stored as `image.jpg`.

* **Replacement Policy**:
  Only one watermark is active. New uploads **replace** old ones.

---

## Example Workflow

1. **Admin uploads watermark**
   → `POST /api/watermark` with `logo.png`
   → stored at `$DSPACE_DIR/watermark/image.png`.

2. **System fetches watermark**
   → `GET /api/watermark`
   → returns raw PNG with correct headers.

3. **Watermark applied**
   → Downstream services (like PDF stamping) fetch this image dynamically from disk.

---

## Error Handling

| Scenario                | Status | Message                   |
| ----------------------- | ------ | ------------------------- |
| Folder missing          | 404    | Not Found                 |
| No watermark uploaded   | 204    | No Content                |
| File missing in request | 400    | Missing file              |
| IOException during save | 500    | Failed to store watermark |

---

## Security Notes

* Upload endpoint is protected by Spring Security (`@PreAuthorize`).
* Only users with `ADMIN` or `SYSTEM_ADMIN` role may upload/replace watermarks.
* Retrieval (`GET`) is **public**, so clients can fetch and use it.

---
