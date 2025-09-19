
# Admin Watermark Feature Documentation

## Overview

The **Admin Watermark** module provides administrators the ability to manage watermark images used across the platform. This includes viewing the current watermark and uploading a new one via drag-and-drop or file selection. The feature is implemented as an Angular module, integrating HTTP communication with a backend watermark API.

---

## Module Structure

### Routing

The watermark page is accessible through the route:

```ts
{
  path: 'admin-watermark',
  component: AdminPannelComponent,
  canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard]
}
```

**Guards:**

* `authenticatedGuard`: Ensures the user is logged in.
* `endUserAgreementCurrentUserGuard`: Ensures the user has accepted the end-user agreement.

---

### AdminPannelModule

**File:** `admin-pannel.module.ts`

```ts
@NgModule({
  declarations: [AdminPannelComponent],
  imports: [
    CommonModule,
    FormsModule,
    HttpClientModule
  ],
  exports: [AdminPannelComponent] 
})
export class AdminPannelModule {}
```

**Responsibilities:**

* Declares `AdminPannelComponent`.
* Provides necessary Angular modules for forms and HTTP communication.

---

### AdminPannelComponent

**File:** `admin-pannel.component.ts`

**Key Features:**

1. **Display Current Watermark**

   * Uses `WatermarkApiService.getCurrent()` to fetch the current watermark.
   * Displays the image if available; otherwise shows a placeholder.

2. **Upload New Watermark**

   * Supports **file picker** and **drag-and-drop** upload.
   * Upload progress is tracked and displayed with a progress bar.
   * On successful upload, the displayed watermark is refreshed.

3. **Drag-and-Drop Support**

   * Handles `dragover`, `dragleave`, and `drop` events.
   * Provides visual feedback when a file is dragged over the dropzone.

4. **Cleanup**

   * Revokes object URLs on component destruction to prevent memory leaks.

**Core Methods:**

* `refresh()`: Loads the current watermark from backend.
* `onPick(event)`: Handles manual file selection.
* `onDragOver(event)`, `onDragLeave(event)`, `onDrop(event)`: Handle drag-and-drop logic.
* `uploadFile(file)`: Uploads the selected file using `WatermarkApiService`.

---

### WatermarkApiService

**File:** `admin-pannel.service.ts`

```ts
@Injectable({ providedIn: 'root' })
export class WatermarkApiService {
  constructor(private http: HttpClient) {}

  getCurrent(): Observable<Blob> {
    return this.http.get(BASE, { responseType: 'blob', withCredentials: true });
  }

  upload(file: File): Observable<HttpEvent<any>> {
    const form = new FormData();
    form.append('file', file);
    const req = new HttpRequest('POST', BASE, form, {
      reportProgress: true,
      withCredentials: true
    });
    return this.http.request(req);
  }
}
```

**Responsibilities:**

* `getCurrent()`: Fetches the current watermark image as a Blob.
* `upload(file)`: Uploads a new watermark file with progress tracking.

---

### Template & Styles

**Template:** `admin-pannel.component.html`

* Divided into two main sections:

  1. **Current Watermark** – displays the current image or placeholder.
  2. **Upload New Watermark** – drag-and-drop area with file input and progress bar.

**Styles:** `admin-pannel.component.scss`

* `.watermark-preview`: Image preview styling.
* `.dropzone`: Drag-and-drop styling including hover effects.
* `.empty-state`: Styling for placeholder when no watermark is present.

---

### Usage Instructions

1. Navigate to `/admin-watermark`.
2. View the currently uploaded watermark.
3. Upload a new watermark using either:

   * Drag-and-drop into the dropzone.
   * Clicking the dropzone and selecting a file.
4. Monitor the progress bar during upload.
5. The current watermark preview refreshes automatically after upload.

**Accepted Formats:** PNG, JPG, WEBP (Recommended: Transparent PNG)

---

### Security & Access Control

* Only authenticated users with accepted end-user agreements can access this feature.
* Backend ensures watermark files are securely handled.

---

### Notes

* Ensure that the backend watermark API is available at `${CURRENT_API_URL}/server/api/watermark`.
* Large image files may affect upload progress responsiveness.
* Transparent PNGs are recommended for proper overlay on documents.

---


