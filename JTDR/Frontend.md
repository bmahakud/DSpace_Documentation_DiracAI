# Frontend Documentation (Angular - JTDR Path)

## Overview
This frontend module provides a UI for managing **JTDR case submissions** within the Angular app.  
It integrates with the backend APIs documented in `backend.md` to handle search, file generation, report download, and submission to JTDR.

The core component is:
- **CnrManagerComponent** (`src/app/cnr-manager/cnr-manager.component.ts`)  
It is linked to the `/jtdr` route in the Angular router and accessible from the top menu (for admins only).

---

## File-by-File Breakdown

### 1. CnrManagerComponent (UI + Logic)
**File:** `src/app/cnr-manager/cnr-manager.component.ts`  
**Template:** `src/app/cnr-manager/cnr-manager.component.html`  
**Styles:** `src/app/cnr-manager/cnr-manager.component.scss`

#### Responsibilities
- Provides **two search modes**:
  - By **CINO number**
  - By **Case (Number / Type / Year)**
- Allows adding search results to a **cart** (zip list).
- Generates **zip files** for selected cart items.
- Lists **Generated Files** with status, actions (submit, check, delete).
- Supports **bulk submission** and **status verification**.
- Provides **report download** (CSV or PDF).
- Offers **filters** and **sorting** for generated files.

#### Key Properties
- `searchMode`: `"cino"` or `"case"`
- `searchResults`: list of files returned from search
- `cartItems`: files selected for zip generation
- `generatedFiles`: files already generated with statuses
- `reportFrom`, `reportTo`, `reportFormat`: report parameters

#### Key Methods
- **Search:**
  - `searchItems()` → fetches search results
  - `onSearchInput()` → live clearing
- **Cart Management:**
  - `addSingleToCart(file)`
  - `removeFromCart(file)`
  - `clearCart()`
  - `generateFiles()` → calls backend to generate zips
- **Generated File Actions:**
  - `submitFile(file)` → submit a single file
  - `submitAllFiles()` → bulk submission + status check
  - `deleteGenerated(file)` → delete generated zip
  - `checkStatus(file)` → check JTDR status
- **Reports:**
  - `downloadReport()` → fetch CSV or PDF report
- **Filters/Sorting:**
  - `applyFilters()` → reload with applied filters

---

### 2. CnrService (API Layer)
**File:** `src/app/cnr-manager/cnr.service.ts`

#### Responsibilities
- Wraps **HTTP calls** to backend APIs
- Provides typed methods for frontend

#### Methods
- `generate(itemUUID: string)` → `POST /export/zip/{itemUUID}`
- `getRecords(page, size, submitted?, sortBy, sortDir)` → `GET /cnr/records`
- `submitCase(cnr: string)` → `POST /jtdr/submit`
- `checkStatus(ackId: string)` → `GET /jtdr/status/{ackId}`
- `getSearchResults(query: string)` → `GET /discover/search/objects?query=`
- `getReportCsv(start, end)` → `GET /jtdr/report` (Accept: CSV)
- `getReportPdf(start, end)` → `GET /jtdr/report` (Accept: PDF)
- `deleteGenerated(fileName: string)` → `DELETE /export/zip/{fileName}`

---

### 3. Module Setup
**Files:**
- `src/app/cnr-manager/cnr-manager.module.ts`
- `src/app/cnr-manager/cnr.module.ts`

Both declare and export `CnrManagerComponent` and import `FormsModule`, `HttpClientModule`, etc.

---

### 4. Routing Integration
**File:** `src/app/app-routing.module.ts` (excerpt)  

```ts
{
  path: "jtdr",
  component: CnrManagerComponent,
  canActivate: [siteAdministratorGuard, endUserAgreementCurrentUserGuard],
}
```

- The component is mounted at `/jtdr` route.
- Access is restricted to **Site Administrators**.

---

### 5. Menu Integration
**File:** `src/app/shared/user-menu/user-menu.component.html`  

```html
<li *ngIf="isAdmin$ | async" class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
  <a class="ds-menu-item" role="menuitem" [routerLink]="['/jtdr']" routerLinkActive="active">
    Jtdr
  </a>
</li>
```

- A new **"Jtdr"** button is shown only for admins.  
- Clicking it navigates to `/jtdr`.

---

## User Flow

1. **Login as Admin** → "Jtdr" menu option appears.
2. Navigate to `/jtdr` → `CnrManagerComponent` UI loads.
3. Search for files:
   - By **CINO** or by **Case details**.
   - Add results to **Cart**.
4. Open **Cart Modal** → "Generate Files".
5. Generated files appear in **Generated Files table** with actions:
   - **Submit** → posts case to JTDR
   - **Check Status** → verifies JTDR response
   - **Delete** → removes generated zip
6. Use **Report section** to download CSV or PDF of activities.
7. Use filters/sorting to refine list.

---

## Potential Improvements
- Add **toasts** instead of alerts for errors.
- Support **multi-select add-to-cart** directly in search results.
- Improve **mobile responsiveness** for tables.
- Add **retry with exponential backoff** for failed API calls.

---

## Summary
The frontend integrates tightly with backend JTDR APIs.  
All user-facing actions (search, generate, submit, check, delete, report) are handled in **CnrManagerComponent**, while **CnrService** abstracts HTTP API calls.  
Access is restricted to **admins** via routing guards and menu visibility.
