

# 🔐 Encrypted PDF Download — Frontend Documentation

## 🎯 Purpose

This feature ensures that when a user downloads a PDF from the viewer, it is passed through the **backend encryption API** before being delivered to the user. This guarantees that files are **not downloaded in raw/original form**, but always as **encrypted PDFs**.

---

## 📂 Key Files & Responsibilities

### 1. **PdfService** (`core/serachpage/pdf-auth.service.ts`)

Handles API communication with the backend.

* **`encryptBitstream(bitstreamId: string): Observable<Blob>`**
  Sends the `bitstreamId` to backend `/server/api/diracai/encrypt-bitstream`.
  Backend responds with an **encrypted PDF blob**.

  ```ts
  encryptBitstream(bitstreamId: string): Observable<Blob> {
    const url = `${CURRENT_API_URL}/server/api/diracai/encrypt-bitstream`;
    return this.http.post(url, { bitstreamId }, {
      responseType: 'blob',
      withCredentials: true
    }).pipe(
      catchError(error => {
        console.error('Encryption API error:', error);
        return throwError(() => new Error('Failed to encrypt file.'));
      })
    );
  }
  ```

* **`createBlobUrl(blob: Blob): string`**
  Converts encrypted blob into a `blob:` URL for download.

* **`revokeBlobUrl(url: string): void`**
  Frees memory by revoking temporary blob URLs after download.

---

### 2. **ViewerComponent** (`viewer.component.ts`)

Implements **download button action**.

* **Download Permission Check**

  ```ts
  if (!this.canDownloadFile) {
    console.warn("Download permission denied");
    return;
  }
  ```

* **Download Method**

  ```ts
  downloadFile(): void {
    const filename = this.generateCustomFilename() || "encrypted.pdf";

    this.pdfService.encryptBitstream(this.currentBitstreamId).subscribe({
      next: (blob) => {
        const blobUrl = this.pdfService.createBlobUrl(blob);
        const a = document.createElement('a');
        a.href = blobUrl;
        a.download = filename;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        this.pdfService.revokeBlobUrl(blobUrl);

        console.log("🔐 Encrypted file downloaded:", filename);
      },
      error: (err) => {
        console.error("❌ Error downloading encrypted file:", err);
      }
    });
  }
  ```

* **Custom Filename Generation**
  Uses metadata (case type, number, year) to generate a structured filename:

  ```ts
  private generateCustomFilename(): string {
    const caseType = this.findMetadataByPartialName("type")
    const caseNumber = this.findMetadataByPartialName("number")
    const caseYear = this.findMetadataByPartialName("year")

    const parts = [caseType, caseNumber, caseYear]
      .map((p) => p?.replace(/[^a-zA-Z0-9]/g, ""))
      .filter(Boolean)

    return parts.length > 0 ? parts.join("_") + ".pdf" : "encrypted.pdf"
  }
  ```

---

### 3. **Viewer Template** (`viewer.component.html`)

Conditional download button in PDF toolbar:

```html
<button *ngIf="canDownloadFile"
        class="icon-button"
        (click)="downloadFile()"
        title="Download PDF"
        aria-label="Download PDF">
  <i class="fas fa-download"></i>
</button>
```

⚠️ If `canDownloadFile = false`, user sees:

```html
<div *ngIf="!canDownloadFile && hasTimeAccess" class="permission-notice download-notice">
  ⚠️ Download not permitted for this file
</div>
```

---

## 🔒 Security Flow (Step-by-Step)

1. User clicks **Download** in toolbar.
2. **Frontend** calls:

   * `ViewerComponent.downloadFile()` →
   * `PdfService.encryptBitstream(bitstreamId)`.
3. **Backend API** returns encrypted PDF blob.
4. **Frontend**:

   * Converts blob → `blob:` URL.
   * Creates hidden `<a>` element with `download` attribute.
   * Triggers download with proper filename.
   * Revokes blob URL (to free memory).
5. **User receives encrypted PDF** (not raw/original).

---

## 📖 Example Download Filenames

* Case Type: `Writ`, Case Number: `123`, Year: `2024`
  → **`Writ_123_2024.pdf`**
* If metadata missing → **`encrypted.pdf`**

---

## ✅ Benefits

* Files are **always encrypted** before leaving the server.
* Prevents raw storage PDFs from being leaked.
* Consistent, human-readable filenames.
* Permissions (`canDownloadFile`) respected.
* Memory-safe with `revokeBlobUrl()`.

