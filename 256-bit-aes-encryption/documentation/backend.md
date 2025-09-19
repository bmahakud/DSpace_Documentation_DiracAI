# 🔐 PDF Encryption & Watermarking — Backend Documentation

## 🎯 Purpose

This controller ensures that PDFs delivered from DSpace are **protected and watermarked** before being downloaded. It provides two main APIs:

* **`/encrypt-bitstream`** → Protects + watermarks PDFs before download.
* **`/decrypt-bitstream`** → Decrypts AES-encrypted files uploaded back to the server.

---

## 📂 Class Location

`org.dspace.app.rest.diracai.controller.EncryptedBitstreamController`

---

## 🔑 Key Responsibilities

1. Retrieve bitstreams from DSpace storage.
2. Log every file access.
3. Apply **image watermark** to each page.
4. Apply **password-based protection**:

   * Owner password (`adminpass`)
   * User password (`userpass`)
5. Restrict user permissions (disable copy, print, modification).
6. Stream back the protected file as **PDF**.
7. Provide a decryption endpoint (AES-based).

---

## ⚙️ Dependencies & Services

* **`BitstreamService`** → Lookup bitstreams by ID.
* **`BitstreamStorageService`** → Retrieve file content streams.
* **`ContextService` / `ContextUtil`** → Manage DSpace context.
* **`FileAccessLogger` + `FileAccessLogService`** → Log downloads for audit trails.
* **`ConfigurationService`** → Locate watermark folder (`dspace.dir/watermark`).
* **`AESUtil`** → Utility for symmetric AES encrypt/decrypt (used in old flow + decrypt API).
* **Apache PDFBox** → Modify PDFs, add watermark, apply password protection.

---

## 📡 API Endpoints

### 1. **Encrypt & Protect Bitstream**

`POST /api/diracai/encrypt-bitstream`

#### Request

```json
{
  "bitstreamId": "UUID_OR_INTERNAL_ID"
}
```

#### Process

1. Validate and retrieve bitstream.
2. Log access (`DOWNLOAD` event).
3. Load PDF with **PDFBox**.
4. Apply watermark (from `dspace.dir/watermark/image.*`).
5. Apply **256-bit AES protection** with:

   * `ownerPassword = "adminpass"`
   * `userPassword = "userpass"`
   * Permissions:

     * ❌ No copy/extract
     * ❌ No modify
     * ❌ No print
6. Return **protected PDF**.

#### Response

* **200 OK** → Protected PDF stream (`application/pdf`)
* **404 Not Found** → Invalid bitstreamId
* **500 Internal Server Error** → Encryption/watermark failure

---

### 2. **Decrypt Uploaded File**

`POST /api/diracai/decrypt-bitstream`

#### Request

* Form-data upload:

  * `file` → Encrypted binary file (from AES encryption flow)

#### Process

1. Read encrypted file.
2. Use `AESUtil.decrypt()` with configured password (`your-256-bit-secret-password`).
3. Return decrypted PDF stream.

#### Response

* **200 OK** → Decrypted PDF
* **500 Internal Server Error** → If AES decryption fails

---

## 🖼️ Watermarking Logic

* Looks for an image in:

  ```
  ${dspace.dir}/watermark/image.*
  ```
* Scales watermark to **40% of page width**.
* Centers watermark on page.
* Sets transparency to **30%** using `PDExtendedGraphicsState`.

Code snippet:

```java
gs.setNonStrokingAlphaConstant(0.3f); // 30% opacity
contentStream.drawImage(pdImage, x, y, imageWidth, imageHeight);
```

---

## 🔒 Security & Restrictions

* **Password Protection**:

  * Owner password → full access
  * User password → limited access
* **Permissions**:

  * Copy/Extract → Disabled
  * Modification → Disabled
  * Printing → Disabled
* **Audit Logging**:

  * Every download logs bitstream ID, action, user context.

---

## 🔧 Error Handling

* **Bitstream not found** → 404.
* **SQL / Context issues** → Logged + rethrown as Runtime.
* **Encryption/Watermark failure** → 500 with stack trace.
* **AES Decryption failure** → 500 with "Decryption failed".

---

## ✅ Benefits

* Protects sensitive PDFs with **watermark + password encryption**.
* Prevents **unauthorized copy, print, or modification**.
* Ensures **all downloads are logged**.
* Supports **AES decryption** for reversible workflows.

---
