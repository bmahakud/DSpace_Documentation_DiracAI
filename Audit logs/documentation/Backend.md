# 📄 Backend Documentation – Audit Logging

---

## 1. 📑 Purpose

The **Audit Logging Module** tracks **user actions, role changes, and file access events** in the system.
It provides APIs for:

* Retrieving **all user sessions** with login/logout details.
* Fetching **detailed audit logs** per user (sessions, role changes, file access).
* Storing **role audit trails**, **session events**, **file access logs**, and **login device audits**.

---

## 2. 📡 API Endpoints

### Base Path:

```
/server/api/audit
```

### Endpoints:

| Method  | Path                            | Description                                  | Returns                 |
| ------- | ------------------------------- | -------------------------------------------- | ----------------------- |
| **GET** | `/api/audit/users`              | Get all users with their latest session info | List of `Audit` DTO     |
| **GET** | `/api/audit/user?userId={uuid}` | Get all audit logs for a specific user       | List of `UserActionLog` |

---

### Example – `/api/audit/users`

Response:

```json
[
  {
    "userId": "e2c1f812-1a56-489d-9abf-11aa22bb33cc",
    "email": "john.doe@example.com",
    "loginTime": "2025-09-20T09:00:00Z",
    "logoutTime": "2025-09-20T10:15:00Z",
    "duration": 4500
  }
]
```

---

### Example – `/api/audit/user?userId={uuid}`

Response:

```json
[
  {
    "action": "LOGIN",
    "timestamp": "2025-09-20T09:00:00Z",
    "ipAddress": "192.168.1.50",
    "userAgent": "Chrome on Windows",
    "objectId": null
  },
  {
    "action": "ROLE_ASSIGN",
    "timestamp": "2025-09-20T09:05:00Z",
    "ipAddress": "192.168.1.50",
    "userAgent": "Chrome on Windows",
    "objectId": null
  },
  {
    "action": "DOWNLOAD",
    "timestamp": "2025-09-20T09:10:00Z",
    "ipAddress": "192.168.1.50",
    "userAgent": "Chrome on Windows",
    "objectId": "case123.pdf"
  }
]
```

---

## 3. 🗂 Entities

### 🔹 **RoleAuditLog** (`role_audit_log` table)

* Tracks **role or permission changes**.
* Columns:

  * `id` (UUID, PK)
  * `acted_by` → who performed the change
  * `affected_user` → user who was affected
  * `action` → `"ROLE_ASSIGN"`, `"PERMISSION_GRANT"`
  * `target` → target group/policy/item
  * `timestamp` → time of change
  * `ip_address`, `user_agent`

---

### 🔹 **UserSessionAudit** (`user_session_audit` table)

* Tracks **login and logout events**.
* Columns:

  * `id` (UUID, PK)
  * `user_id`, `email`
  * `event_type` → `"LOGIN"` / `"LOGOUT"`
  * `session_id`
  * `login_time`, `logout_time`, `duration_seconds`
  * `ip_address`, `user_agent`
  * `timestamp`

---

### 🔹 **FileAccessLog** (`file_access_log` table)

* Tracks **file actions** (view/download).
* Columns:

  * `id` (Long, PK)
  * `file_id`, `file_name`
  * `user_id`, `user_email`
  * `action` → `"VIEW"` / `"DOWNLOAD"`
  * `timestamp`
  * `ip_address`, `user_agent`
  * `suspicious` (boolean)

---

### 🔹 **LoginDeviceAudit** (`login_device_audit` table)

* Tracks **login attempts from devices**.
* Columns:

  * `id` (UUID, PK)
  * `eperson_id`
  * `ip_address`, `user_agent`
  * `device_id`
  * `login_time`
  * `status` → `"SUCCESS"` / `"FAILURE"`
  * `failed_attempts`

---

## 4. 🛠 Services

### 🔹 **UserSessionAuditService**

* Logs **login events**:

  ```java
  log("LOGIN", user, request);
  ```
* Logs **logout events**:

  * Finds latest login record for user + IP.
  * Updates `logout_time` and `duration_seconds`.

---

### 🔹 **AuditServiceImpl**

Implements high-level aggregation for APIs:

* `getAllUsers()`

  * Collects latest session audit for each user.
  * Returns summary DTOs (`Audit`).

* `getByUser(userId)`

  * Fetches logs from:

    * **UserSessionAudit** (LOGIN/LOGOUT).
    * **RoleAuditLog** (ROLE\_ASSIGN, PERMISSION\_GRANT).
    * **FileAccessLog** (VIEW, DOWNLOAD).
  * Converts them into `UserActionLog` DTOs.
  * Sorts by timestamp (chronological).

---

## 5. 📑 Data Transfer Objects (DTOs)

### 🔹 `Audit` (user summary)

```json
{
  "userId": "UUID",
  "email": "string",
  "loginTime": "timestamp",
  "logoutTime": "timestamp",
  "duration": "number (seconds)"
}
```

### 🔹 `UserActionLog` (detailed logs)

```json
{
  "action": "LOGIN / LOGOUT / ROLE_ASSIGN / PERMISSION_GRANT / VIEW / DOWNLOAD",
  "timestamp": "timestamp",
  "ipAddress": "string",
  "userAgent": "string",
  "objectId": "string | null"
}
```

---

## 6. 🔍 Flow

1. **User Logs In** → `UserSessionAuditService.log("LOGIN")` → entry in `user_session_audit`.
2. **User Logs Out** → `logLogout()` updates logout time + duration.
3. **Admin Changes Role** → `RoleAuditLog` entry created.
4. **User Downloads/View File** → `FileAccessLog` entry created.
5. **Device Login Attempt** → `LoginDeviceAudit` entry created.
6. **Frontend Calls `/api/audit/...`** → Aggregated view returned via `AuditServiceImpl`.

---

## 7. ✅ Features

* Centralized audit trail across **sessions, roles, file access, devices**.
* API for **summary view** (users list).
* API for **detailed per-user timeline**.
* Tracks **IP + device fingerprinting**.
* Supports **suspicious activity detection** via `suspicious` flag.
* Extensible → new audit event types can be added.

---

## 8. 🔐 Security Considerations

* Audit logs are **read-only via API**.
* Logged automatically by backend (cannot be faked by clients).
* Requires **authenticated + authorized access** on frontend.

---

