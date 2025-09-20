# 📄 Bitstream Viewer Documentation (Angular Frontend)

---

## 1. 📂 File Locations

| File                               | Location in Angular Project                                              |
| ---------------------------------- | ------------------------------------------------------------------------ |
| `viewer.component.ts`              | `src/app/item-page/field-components/view-file-pdf/viewer.component.ts`   |
| `viewer.component.html`            | `src/app/item-page/field-components/view-file-pdf/viewer.component.html` |
| `viewer.component.scss`            | `src/app/item-page/field-components/view-file-pdf/viewer.component.scss` |
| `viewer.module.ts`                 | `src/app/item-page/field-components/view-file-pdf/viewer.module.ts`      |
| `bitstream-comment.service.ts`     | `src/app/core/serachpage/bitstream-comment.service.ts`                   |
| `bitstream-permissions.service.ts` | `src/app/core/serachpage/bitstream-permissions.service.ts`               |
| `pdf-auth.service.ts` (PdfService) | `src/app/core/serachpage/pdf-auth.service.ts`                            |

---

## 2. 📝 Complete Code

You already have **complete code** for all files. ✅
I’ll not repeat them fully here since you pasted them — instead I’ll **annotate line-by-line explanations**.

---

## 3. ✅ Feature List (All Files Combined)

### ViewerComponent

* 📑 Metadata panel with ordered fields
* 📖 PDF viewer with search, zoom, navigation, print, download
* 🖼 Image viewer with zoom, fullscreen, download
* 🎥 Video viewer with playback, error handling, download
* 🎵 Audio viewer with playback, download
* 💬 Comment panel with add/delete (admin only)
* 🔐 Permissions (time-based, print/download restrictions)
* ⏳ Auto-expiry and periodic re-check

### BitstreamCommentService

* Fetch comments for bitstream
* Add new comment
* Update comment
* Delete comment

### BitstreamPermissionsService

* Fetch permissions (policies, admin flag)
* Check print/download rights
* Validate time-based access (start/end date checks)
* Auto-format user-friendly messages (e.g., “Access expires in 3 hours”)

### PdfService

* Secure fetch for restricted PDFs
* Encrypt bitstream before download
* Create/revoke Blob URLs

---

## 4. 🔍 Feature-wise Code Explanations

---

### 📑 **Metadata Panel**

* `fetchMetadataFromApi()` in `viewer.component.ts`
  Fetches metadata, excludes internal fields, arranges keys in human-readable order.

### 📖 **PDF Viewer**

* Uses `pdfjs-dist` to render PDFs securely.
* `renderAllPages()` loops pages, renders canvas + text layers for search.
* Search implemented with regex-based highlight + navigation.

### 🖼 **Image Viewer**

* `<img>` with zoom and fullscreen.
* Downloads are secured using `<canvas>` re-render (prevents direct file steal).

### 🎥 **Video Viewer**

* `<video>` with error handling.
* Shows debug info if load fails.

### 🎵 **Audio Viewer**

* `<audio>` with secure download.

### 💬 **Comments**

* `bitstreamCommentService.getComments()` fetches all.
* Admins: `addComment()` and `deleteComment()`.
* Confirmation modal prevents accidental deletion.

### 🔐 **Permissions**

* `getBitstreamPermissions()` fetches policies.
* Admins bypass all checks.
* Normal users → must respect policy `startDate`/`endDate`.
* `canDownload()` and `canPrint()` return booleans.
* `checkTimeAccess()` returns message + expiry times.

### ⏳ **Auto-Expiry**

* `setupAccessExpirationTimer()` sets timeout until expiry.
* `setupPeriodicAccessCheck()` checks every 60s for revocation.

---

## 5. 📖 Line-by-Line Explanations

I’ll give you **example detailed explanation** for one file.
Let’s do `bitstream-permissions.service.ts` since it’s most critical.

---

### 📂 `src/app/core/serachpage/bitstream-permissions.service.ts`

```ts
@Injectable({ providedIn: "root" })
export class BitstreamPermissionsService {
  private baseUrl = `${CURRENT_API_URL}/server/api/custom/bitstreams`
```

* `@Injectable({ providedIn: "root" })`: Makes service available app-wide without extra module imports.
* `baseUrl`: Common API prefix for all bitstream permission endpoints.

---

```ts
getBitstreamPermissions(bitstreamId: string): Observable<BitstreamPermission> {
  return this.http.get<BitstreamPermission>(`${this.baseUrl}/${bitstreamId}/permissions`).pipe(
    catchError((error) => {
      console.error(`Error fetching permissions for bitstream ${bitstreamId}:`, error)
      return of({ userId: "", policies: [], bitstreamId })
    }),
  )
}
```

* Calls backend to fetch permissions for a specific bitstream.
* Returns `{userId, policies, bitstreamId}` even if API fails → avoids breaking app.

---

```ts
hasBitstreamPermissions(bitstreamId: string): Observable<boolean> {
  return this.getBitstreamPermissions(bitstreamId).pipe(
    map((permission) => {
      return permission.isAdmin === true || (permission.policies && permission.policies.length > 0)
    }),
  )
}
```

* Checks if user has **any permission** or is admin.

---

```ts
canDownload(bitstreamId: string): Observable<boolean> {
  return this.getBitstreamPermissions(bitstreamId).pipe(
    map((permission) => {
      if (permission.isAdmin === true) return true
      if (!permission.policies || permission.policies.length === 0) return false
      const now = new Date()
      return permission.policies.some((policy) => policy.download && this.isWithinTimeRange(policy, now))
    }),
  )
}
```

* ✅ Admins always allowed.
* ✅ Non-admins allowed only if policy says `download: true` AND current time within policy window.

---

```ts
canPrint(bitstreamId: string): Observable<boolean> {
  return this.getBitstreamPermissions(bitstreamId).pipe(
    map((permission) => {
      if (permission.isAdmin === true) return true
      if (!permission.policies || permission.policies.length === 0) return false
      const now = new Date()
      return permission.policies.some((policy) => policy.print && this.isWithinTimeRange(policy, now))
    }),
  )
}
```

* Same logic as download, but checks `print: true`.

---

```ts
checkTimeAccess(bitstreamId: string): Observable<TimeAccessStatus> {
  return this.getBitstreamPermissions(bitstreamId).pipe(
    map((permission) => {
      if (permission.isAdmin === true) {
        return { hasAccess: true, message: "You have admin access to this file.", validUntil: null, validFrom: null }
      }
      ...
    }),
  )
}
```

* Builds **user-friendly status message** (e.g., “Access expired yesterday” / “Available from tomorrow”).
* Handles cases:

  * Admin → full access.
  * No policies → denied.
  * Valid policy → access granted until `endDate`.
  * Expired policy → message with expired date.
  * Future policy → message with start date.

---

```ts
private isWithinTimeRange(policy: BitstreamPolicy, now: Date): boolean {
  if (policy.startDate && now < new Date(policy.startDate)) return false
  if (policy.endDate && now > new Date(policy.endDate)) return false
  return true
}
```

* Utility function to validate policy timeframe.

---

```ts
private getTimeRemainingText(now: Date, endDate: Date): string {
  const diffMs = endDate.getTime() - now.getTime()
  ...
}
```

* Converts raw milliseconds to human-friendly countdown:

  * `"2 days 5 hours remaining"`
  * `"15 minutes remaining"`

---



```ts
import { Injectable } from '@angular/core';
```

➡ Imports `Injectable` so Angular knows this is a service.

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
```

➡ `HttpClient` is used for making API requests.
➡ `HttpHeaders` lets us send headers (like `Content-Type`).

```ts
import { Observable } from 'rxjs';
```

➡ `Observable` is Angular’s way to handle async data streams (API responses).

```ts
import { CURRENT_API_URL } from './api-urls';
```

➡ Imports the **base API URL** constant (keeps endpoints consistent).

---

```ts
export interface BitstreamComment {
  id?: number;
  bitstreamId: string;
  comment?: string; // request field
  text?: string;    // response field
}
```

➡ Defines the **comment model**.

* `id?`: optional (only exists after backend creates it).
* `bitstreamId`: UUID of the file.
* `comment`: text when sending request.
* `text`: response field (backend returns saved text).

---

```ts
@Injectable({ providedIn: 'root' })
export class BitstreamCommentService {
  private baseUrl = `${CURRENT_API_URL}/server/api/bitstream/comment`;
```

➡ Declares the service as globally available.
➡ Defines the API endpoint for comments.

---

```ts
constructor(private http: HttpClient) {}
```

➡ Injects Angular’s `HttpClient` into the service.

---

```ts
getComments(bitstreamId: string): Observable<BitstreamComment[]> {
  return this.http.get<BitstreamComment[]>(`${this.baseUrl}/bitstream/${bitstreamId}`, {
    withCredentials: true
  });
}
```

➡ `GET /bitstream/{id}` → Fetch all comments.
➡ `withCredentials: true` ensures user’s auth session is included.

---

```ts
addComment(comment: BitstreamComment): Observable<BitstreamComment> {
  return this.http.post<BitstreamComment>(this.baseUrl, comment, {
    withCredentials: true
  });
}
```

➡ `POST /comment` → Add a new comment.
➡ Request body contains `{ bitstreamId, comment }`.

---

```ts
updateComment(id: number, newText: string): Observable<BitstreamComment> {
  const headers = new HttpHeaders({ 'Content-Type': 'text/plain' });
  return this.http.put<BitstreamComment>(`${this.baseUrl}/${id}`, newText, {
    headers,
    withCredentials: true
  });
}
```

➡ `PUT /comment/{id}` → Update an existing comment.
➡ Sends raw `text/plain` instead of JSON.

---

```ts
deleteComment(id: number): Observable<any> {
  return this.http.delete(`${this.baseUrl}/${id}`, {
    withCredentials: true
  });
}
```

➡ `DELETE /comment/{id}` → Remove a comment.

---

---

## 2. 📂 File: `bitstream-permissions.service.ts`

**Location:**

```
src/app/core/serachpage/bitstream-permissions.service.ts
```

---

### ✅ Features

* Fetch bitstream permissions
* Check if user is admin
* Check **download** and **print** rights
* Check **time-based access** (startDate, endDate)
* Format human-readable messages like “Access expires in 2 hours”

---

### 📝 Code with Explanations

We already started this one. Example:

```ts
export interface BitstreamPermission {
  userId: string
  policies: BitstreamPolicy[]
  bitstreamId: string
  isAdmin?: boolean // Added isAdmin flag
}
```

➡ `isAdmin` = true → user bypasses all restrictions.

---

```ts
canDownload(bitstreamId: string): Observable<boolean> {
  return this.getBitstreamPermissions(bitstreamId).pipe(
    map((permission) => {
      if (permission.isAdmin === true) {
        return true
      }
      ...
    }),
  )
}
```

➡ Logic: Admin → always true.
➡ Otherwise, only if a policy with `download: true` exists **AND** time is valid.

---

```ts
checkTimeAccess(bitstreamId: string): Observable<TimeAccessStatus> { ... }
```

➡ Returns **user-friendly access status**.
Examples:

* “You have access until Sep 21, 5:30 PM (2 hours remaining)”
* “Access expired on Sep 19”
* “Access will be available from Sep 22”

---

---

## 3. 📂 File: `pdf-auth.service.ts` (PdfService)

**Location:**

```
src/app/core/serachpage/pdf-auth.service.ts
```

---

### ✅ Features

* Fetch restricted PDF securely
* Encrypt bitstream before download
* Create/revoke blob URLs

---

### 📝 Code with Explanations

```ts
fetchRestrictedPdf(bitstreamUuid: string): Observable<Blob> {
  const url = `${CURRENT_API_URL}/server/api/custom/bitstreams/${bitstreamUuid}/filtered-content`;
  return this.http.get(url, { 
    responseType: 'blob', 
    withCredentials: true 
  }).pipe(
    retry(2),
    catchError(error => {
      console.error('Error fetching PDF:', error);
      return throwError(() => new Error('Failed to fetch PDF. Please try again later.'));
    })
  );
}
```

➡ Calls secure endpoint → backend applies filters (watermark, restrictions).
➡ Returns **Blob** → can be rendered in PDF.js.
➡ Retries request twice if it fails.

---

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

➡ Calls encryption API → generates encrypted PDF.
➡ Returns a downloadable blob.

---

```ts
revokeBlobUrl(url: string): void {
  if (url && url.startsWith('blob:')) {
    URL.revokeObjectURL(url);
  }
}
```

➡ Cleans up memory after Blob URLs are no longer needed.

---

---

## 4. 📂 File: `viewer.component.ts`

**Location:**

```
src/app/item-page/field-components/view-file-pdf/viewer.component.ts
```

---

### ✅ Features

* Metadata panel (custom order, collapsible)
* PDF viewer (render, zoom, search, print, download)
* Image viewer (zoom, fullscreen, secure download)
* Video viewer (error handling, secure playback)
* Audio viewer (secure playback & download)
* Comments (admin add/delete)
* Permissions (download/print/time)
* Expiry timer & periodic checks

---

### 📝 Line Explanations (Highlights)

```ts
ngOnInit(): void {
  this.itemUuid = this.route.snapshot.paramMap.get('itemUuid')!
  this.bitstreamUuid = this.route.snapshot.paramMap.get('bitstreamUuid')!
```

➡ Extracts file UUIDs from route (`/i/:itemUuid/f/:bitstreamUuid`).

---

```ts
this.permissionsService.checkTimeAccess(this.bitstreamUuid).subscribe(status => {
  this.accessStatus = status
  if (status.hasAccess) {
    this.loadFileBasedOnType()
  }
})
```

➡ Checks permissions before loading file.
➡ If denied → shows error message instead of loading file.

---

```ts
renderAllPages(pdf: PDFDocumentProxy): void {
  for (let pageNum = 1; pageNum <= pdf.numPages; pageNum++) {
    this.renderPage(pdf, pageNum)
  }
}
```

➡ Loops through all PDF pages → renders into `<canvas>`.

---

```ts
highlightSearchMatches(textContent: string, searchQuery: string): void {
  const regex = new RegExp(searchQuery, 'gi')
  ...
}
```

➡ Implements **text highlight** in PDF.

---

```ts
setupAccessExpirationTimer(expiryDate: Date): void {
  const now = new Date()
  const timeout = expiryDate.getTime() - now.getTime()
  if (timeout > 0) {
    this.expiryTimer = setTimeout(() => {
      this.accessStatus.hasAccess = false
    }, timeout)
  }
}
```

➡ Auto-blocks file once policy expires.

---

```ts
addComment(): void {
  if (this.newComment.trim() === '') return
  const comment: BitstreamComment = { bitstreamId: this.bitstreamUuid, comment: this.newComment }
  this.commentService.addComment(comment).subscribe(() => this.loadComments())
}
```

➡ Admins can add new comments.

---

```ts
confirmDeleteComment(commentId: number): void {
  if (confirm('Are you sure you want to delete this comment?')) {
    this.commentService.deleteComment(commentId).subscribe(() => this.loadComments())
  }
}
```

➡ Confirmation before deleting comment.

---

---
