# 📄 Frontend Documentation – User Audit Module

---

## 1. 📂 File Locations

| File                                              | Location               |
| ------------------------------------------------- | ---------------------- |
| `admin-audit.module.ts`                           | `src/app/admin-audit/` |
| `admin-audit-routing.module.ts`                   | `src/app/admin-audit/` |
| `admin-audit.service.ts`                          | `src/app/admin-audit/` |
| `audit-user-list.component.ts / .html / .scss`    | `src/app/admin-audit/` |
| `audit-user-details.component.ts / .html / .scss` | `src/app/admin-audit/` |

---

## 2. 📑 Module Overview

The **User Audit Module** allows administrators to:

* View all users with summary details (login/logout/duration).
* Drill down into an individual user’s **audit trail**.
* Display device usage and action timeline.

---

## 3. 📡 Routing

### Base Path:

```
/user-audit
```

### Routes:

| Path                       | Component                   | Purpose                                    |
| -------------------------- | --------------------------- | ------------------------------------------ |
| `/user-audit`              | `AuditUserListComponent`    | Displays all users’ audit summary          |
| `/user-audit/user/:userId` | `AuditUserDetailsComponent` | Displays detailed logs for a specific user |

---

## 4. 🧩 Components

### 🔹 4.1 `AuditUserListComponent`

**File:** `audit-user-list.component.ts`

* Fetches **all users’ summary audit logs** from backend.
* Displays in a **table** with columns:

  * Email
  * Last Login
  * Last Logout
  * Duration
* Clicking a row navigates to `/user-audit/user/:userId`.

**Template (`.html`):**

* Renders a table of users with `*ngFor`.
* Each row clickable → navigates to details page.

**Styles (`.scss`):**

* Table styling (hover effects, clickable rows).
* Email styled with underlined blue link.

---

### 🔹 4.2 `AuditUserDetailsComponent`

**File:** `audit-user-details.component.ts`

* Fetches **audit logs for a specific user** by `userId`.
* Extracts unique devices used (`userAgent`).
* Displays:

  * Devices used (summary box).
  * Full action timeline in a table (Action, Timestamp, Object, IP, Device).

**Template (`.html`):**

* `summary-box`: lists all unique devices.
* `table`: displays action timeline.

**Styles (`.scss`):**

* Highlight devices summary box.
* Table with borders, hover effects.

---

## 5. ⚙️ Service

### 🔹 `AdminAuditService`

**File:** `admin-audit.service.ts`

Provides methods for HTTP calls to backend:

| Method                       | Endpoint                                     | Purpose                                   |
| ---------------------------- | -------------------------------------------- | ----------------------------------------- |
| `getAllUsers()`              | `GET /server/api/audit/users`                | Fetch all users with login/logout summary |
| `getAuditLogsByUser(userId)` | `GET /server/api/audit/user?userId={userId}` | Fetch audit trail for specific user       |

Uses Angular’s `HttpClient` + `Observable` for async calls.

---

## 6. 🛠 Workflow

1. **User navigates** to `/user-audit`.

   * `AuditUserListComponent` loads → calls `getAllUsers()`.
   * Displays all users with login/logout details.

2. **User clicks** a row.

   * Router navigates to `/user-audit/user/:userId`.

3. **`AuditUserDetailsComponent` loads**.

   * Extracts `userId` from route params.
   * Calls `getAuditLogsByUser(userId)`.
   * Displays device summary + action timeline.

---

## 7. 📖 Example Data Flow

### Response – `/server/api/audit/users`

```json
[
  {
    "userId": "123",
    "email": "user1@example.com",
    "loginTime": "2025-09-20T09:00:00Z",
    "logoutTime": "2025-09-20T10:00:00Z",
    "duration": "1h"
  }
]
```

### Response – `/server/api/audit/user?userId=123`

```json
[
  {
    "action": "LOGIN",
    "timestamp": "2025-09-20T09:00:00Z",
    "objectId": null,
    "ipAddress": "192.168.1.100",
    "userAgent": "Chrome on Windows"
  },
  {
    "action": "VIEW_ITEM",
    "timestamp": "2025-09-20T09:10:00Z",
    "objectId": "item-456",
    "ipAddress": "192.168.1.100",
    "userAgent": "Chrome on Windows"
  }
]
```

---

## 8. ✅ Key Features

* **Two-level navigation:** summary → detailed audit logs.
* **Unique device detection:** highlights devices used by user.
* **Timeline view:** actions with timestamps and metadata.
* **Responsive UI:** hover highlights + clickable rows.

---

## 9. 🔐 Guards & Access

* Routes (`/user-audit`) are protected by:

  ```ts
  canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard]
  ```

  → Only authenticated users with agreement signed can access.

---
