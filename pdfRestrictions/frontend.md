Got it ✅ — I’ll prepare a **clear frontend documentation** for your **PDF Viewer** module (Angular + PDF.js).
This will read like the `backend.md` and `frontend.md` docs you already have in your repo.

---

# 📘 PDF Viewer Frontend Documentation

## Purpose

The **PDF Viewer** provides a secure and feature-rich way to view restricted documents in the Angular frontend. It integrates with the backend API to fetch restricted bitstreams, enforce permissions (download/print), and apply time-based access control.

---

## 🔑 Key Features

* **Restricted Access**: Fetches PDFs through the backend API (`/bitstreams/:uuid/filtered-content`) with credentials.
* **Time-based Access**: Restricts visibility with `validUntil` expiration messages.
* **Permissions**: Enforces `canDownloadFile` and `canPrintFile`.
* **PDF Toolbar**:

  * Page navigation (`prevPage`, `nextPage`)
  * Zoom (`zoomIn`, `zoomOut`)
  * Search within PDF
  * Print & Download (conditional)
* **Panels**:

  * Left Metadata Panel (toggleable)
  * Right Comments Panel (toggleable, admin-only actions)
* **Multiple Formats**: Supports PDF, image, video, and audio viewers, selected dynamically based on file type.

---

## 📂 File Structure

```
item-page/
 └── field-components/
      └── view-file-pdf/
          ├── viewer.component.ts    # Angular logic & PDF.js integration
          ├── viewer.component.html  # UI layout (PDF, image, video, audio)
          ├── viewer.component.scss  # Styles
          ├── viewer.module.ts       # Module wrapper
core/
 └── serachpage/
      └── pdf-auth.service.ts        # PdfService for API requests
```

---

## 🧩 Main Components

### 1. **Routes**

* `app-routing.module.ts`

  ```ts
  {
    path: "viewer",
    loadChildren: () => import("./item-page/simple/field-components/viewer-routes")
      .then((m) => m.ROUTES),
    canActivate: [authenticatedGuard],
  }
  ```
* `viewer-routes.ts`

  ```ts
  { path: 'i/:itemUuid/f/:bitstreamUuid', component: ViewerComponent }
  ```

👉 Navigating to `/viewer/i/<itemUuid>/f/<bitstreamUuid>` loads the `ViewerComponent`.

---

### 2. **ViewerComponent**

Located in `viewer.component.ts`.

Handles:

* PDF rendering via **PDF.js** (`pdfjsLib.getDocument()`).
* State management:

  * `isPdfFile`, `isImageFile`, `isVideoFile`, `isAudioFile`
  * Permissions: `canDownloadFile`, `canPrintFile`
  * Time-based access: `hasTimeAccess`, `timeAccessStatus`
* Toolbar actions:

  * `prevPage()`, `nextPage()`
  * `zoomIn()`, `zoomOut()`
  * `toggleSearch()`, `searchPdf()`, `clearSearch()`
  * `printFile()`, `downloadFile()`
* Metadata panel (left) and comments panel (right).

**Template Reference**

```ts
@ViewChild("pdfContainer", { static: false }) pdfContainer!: ElementRef<HTMLDivElement>
```

* The `#pdfContainer` in HTML is the **target container** where PDF.js injects rendered pages.

---

### 3. **PdfService**

Located in `core/serachpage/pdf-auth.service.ts`.

Responsibilities:

* Fetch restricted PDFs from the backend API:

  ```ts
  fetchRestrictedPdf(bitstreamUuid: string): Observable<Blob>
  ```
* Create and revoke blob URLs for safe memory handling.

**Example Usage in ViewerComponent**

```ts
this.pdfService.fetchRestrictedPdf(bitstreamUuid).subscribe(blob => {
  const blobUrl = this.pdfService.createBlobUrl(blob)
  this.fileUrl = blobUrl
  this.loadPdf()
})
```

---

### 4. **HTML Template (viewer.component.html)**

* **Container**: `<div class="viewer-container">`
* **PDF Viewer**:

  ```html
  <div class="pdf-container" *ngIf="isPdfFile && hasTimeAccess">
    <div class="toolbar pdf-toolbar-header"> ... </div>
    <div class="pdf-scroll-container" #pdfContainer></div>
  </div>
  ```
* **Other formats**: Conditional blocks for images, video, and audio.
* **Panels**: Metadata (left), Comments (right).
* **Access Notices**: Time restriction banners + permission warnings.

---

## ⚙️ How it Works (Flow)

1. User navigates to `/viewer/i/:itemUuid/f/:bitstreamUuid`.
2. `ViewerComponent` loads, calls `fetchRestrictedPdf()`.
3. Backend returns a restricted blob (PDF).
4. Blob converted to `blob:` URL → assigned to `fileUrl`.
5. `loadPdf()` uses **PDF.js** to load and render into `#pdfContainer`.
6. Toolbar buttons call methods (`zoomIn`, `nextPage`, etc.) to re-render pages dynamically.
7. Metadata and comments fetched from APIs and displayed in side panels.

---

## 🔒 Security & Access Control

* **API Layer**: `fetchRestrictedPdf()` uses `withCredentials: true`.
* **UI Layer**:

  * Access banner: `Access expires: {{ timeAccessStatus.validUntil }}`
  * Conditional buttons:

    ```html
    <button *ngIf="canDownloadFile" ...>Download</button>
    <button *ngIf="canPrintFile" ...>Print</button>
    ```
* **Comment Permissions**: Only `isAdmin` can add or delete comments.

---

## 📖 Example Usage

To open a file:

1. Navigate to:

   ```
   /viewer/i/1234-fake-item-uuid/f/abcd-fake-bitstream-uuid
   ```
2. If `isPdfFile = true`, the PDF loads in `#pdfContainer`.
3. Use toolbar to navigate, zoom, or search.
4. If access expired → shows "Access Restricted" notice.

---

✅ This documentation gives a **full overview of the frontend PDF Viewer module** (routing, component logic, service layer, HTML layout, and security).

Do you also want me to prepare a **developer onboarding guide** (step-by-step: how to add a new viewer type, e.g. Word/Excel) or just keep this as module documentation?
