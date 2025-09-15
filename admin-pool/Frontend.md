
# AdminPool Module - Frontend Documentation

## 📘 Purpose
The **Admin Pool** feature in Angular provides administrators with a dashboard to manage bulk-upload requests.  
It integrates with backend APIs to:
- View pooled, claimed, rejected, and accepted bulk-upload batches.
- Review the files inside a batch.
- Approve or reject submissions.
- Track uploader, reviewer, collection, and audit dates.

---

## 🔹 Files Created / Modified

- `admin-pool.module.ts` → Declares the feature module.
- `admin-pool-routing.module.ts` → Sets up routing for this module.
- `admin-pool.component.ts` → Component handling UI logic.
- `admin-pool.component.html` → Template rendering task lists and review dialogs.
- `admin-pool.component.scss` → Styling for the UI.
- `admin-service.ts` (AdminPoolService) → Service to interact with backend APIs.

---

## 🔹 Backend API Integration

Base: `CURRENT_API_URL + /server/api/bulk-upload`

- **GET** `/status/{status}` → Fetch batches by status (`CLAIMED`, `APPROVED`, `REJECTED`).
- **GET** `/pooled` → Fetch pooled tasks for current reviewer/admin.
- **GET** `/{batchId}` → Fetch files and metadata inside a batch.
- **POST** `/approve/{uuid}` → Approve a batch (Admin + Reviewer required).
- **POST** `/reject/{uuid}` → Reject a batch (Admin + Reviewer required).

---

## 🔹 File Explanations

### `admin-pool.module.ts`
- Declares `AdminPoolComponent`.
- Imports `CommonModule`, `FormsModule`, and `AdminPoolRoutingModule`.

### `admin-service.ts` (AdminPoolService)
Centralized service for backend interaction:
- `fetchBatches(status)` → Get tasks by status.
- `getBatchFiles(batchId)` → Get batch details + files.
- `approve(uuid, collectionUuid)` → Approves request with zip + collection params.
- `reject(uuid)` → Rejects batch.
- `getPooledTasks()` → Gets pooled tasks for reviewer/admin.
- `getAcceptedSubmissions()` → Fetches approved submissions.
- `getRejectedSubmissions()` → Fetches rejected submissions.

### `admin-pool.component.ts` (AdminPoolComponent)
Controls the UI logic:
- Loads claimed, pooled, rejected, and accepted tasks on init.
- Opens review dialog with `openReviewDialog(batch)`.
- Approves/Rejects submissions and refreshes data.
- Handles accepted submissions view.

State Variables:
- `claimedTasks`, `pooledTasks`, `rejectedTasks`, `acceptedSubmissions`.
- `selectedBatch`, `dummyFiles`, `reviewLoading`, `showAccepted`.

### `admin-pool.component.html`
Defines UI with tables and actions:
- Rejected tasks table.
- Claimed tasks with “Perform Task”.
- Pooled tasks with “Show Task”.
- Review dialog for batch → Approve / Reject / Cancel.
- Accepted submissions view with nested file details.

### `admin-pool.component.scss`
Custom styling:
- `.admin-pool` wrapper for consistent layout.
- Styled tables, buttons, review section, accepted submissions section.

### `admin-pool-routing.module.ts`
Defines route for this feature:
```ts
const routes: Routes = [
  { path: '', component: AdminPoolComponent }
];
```

---

## 🔹 Logic Flow

1. Admin enters **AdminPool** → loads claimed, pooled, rejected, and accepted tasks.
2. Tables display tasks grouped by status.
3. Clicking “Perform Task” → Opens review dialog → Displays files in batch.
4. Admin chooses **Approve** or **Reject** → Backend updates status → Lists refresh.
5. Admin can view **Accepted Submissions** in a separate section.

---

## ✅ Next Steps

- Add authentication guards to protect route access (Admins only).
- Improve error handling with Angular Material Snackbars.
- Add pagination for large task lists.

---
