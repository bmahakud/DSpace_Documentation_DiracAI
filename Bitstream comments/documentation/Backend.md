# 📄 Backend Documentation – Bitstream Comments

---

## 1. 📂 File Locations

| File                               | Location                                                                       |
| ---------------------------------- | ------------------------------------------------------------------------------ |
| `BitstreamCommentController.java`  | `dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/`   |
| `BitstreamCommentService.java`     | `dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/`      |
| `BitstreamCommentServiceImpl.java` | `dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/impl/` |
| `BitstreamCommentRepository.java`  | `dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/Repository/`   |
| `BitstreamComment.java` (Entity)   | `dspace-server-webapp/src/main/java/org/dspace/content/Diracai/`               |

---

## 2. 📝 API Endpoints

### Base Path:

```
/api/bitstream/comment
```

### Endpoints:

| Method     | Endpoint                                         | Purpose                                     |
| ---------- | ------------------------------------------------ | ------------------------------------------- |
| **POST**   | `/api/bitstream/comment`                         | Create a new comment (Admin only)           |
| **PUT**    | `/api/bitstream/comment/{id}`                    | Update an existing comment                  |
| **DELETE** | `/api/bitstream/comment/{id}`                    | Soft delete a comment (`is_deleted = true`) |
| **GET**    | `/api/bitstream/comment/{id}`                    | Get a comment by ID                         |
| **GET**    | `/api/bitstream/comment/bitstream/{bitstreamId}` | Get all comments for a specific bitstream   |

---

### Example Requests & Responses

#### **POST /api/bitstream/comment**

Request:

```json
{
  "comment": "This is an admin comment.",
  "bitstreamId": "eaf3d2a1-1234-4bcd-9876-ab12cd34ef56"
}
```

Response:

```json
{
  "id": 1,
  "commentDate": "2025-09-20T12:34:56",
  "text": "This is an admin comment.",
  "deleted": false,
  "commenterName": "Admin User"
}
```

---

#### **GET /api/bitstream/comment/bitstream/{bitstreamId}**

Response:

```json
[
  {
    "id": 1,
    "commentDate": "2025-09-20T12:34:56",
    "text": "This is an admin comment.",
    "deleted": false,
    "commenterName": "Admin User"
  }
]
```

---

## 3. ✅ Features

* 🔐 **Role-based access** → Only **admin users** can create comments.
* 📥 **CRUD support** → Create, Read, Update, Delete (soft delete).
* 📑 **Metadata included** → Commenter’s name, comment date.
* 🗑 **Soft delete** → Marks comments as deleted (`isDeleted=true`) instead of removing.
* 🕒 **Auto timestamp** → Each comment stores creation time.
* 🔄 **DTO-based design** → Uses Request/Response DTOs for clean API layer.

---

## 4. 🔍 Feature-wise Explanation

### 🔹 **Entity (`BitstreamComment`)**

* Represents a database table `bitstream_comment`.
* Fields: `id, text, bitstreamId, commenterId, commentDate, isDeleted`.
* `@PrePersist` ensures `commentDate` is set automatically.

---

### 🔹 **Repository (`BitstreamCommentRepository`)**

* Extends `JpaRepository` → provides CRUD automatically.
* Adds custom query:

  ```java
  List<BitstreamComment> findByBitstreamIdAndIsDeletedFalse(UUID bitstreamId);
  ```

  → Fetches **only active (non-deleted) comments**.

---

### 🔹 **Service Interface (`BitstreamCommentService`)**

Defines business logic methods:

* `create`, `update`, `delete`, `getById`, `getByBitstreamId`.

---

### 🔹 **Service Implementation (`BitstreamCommentServiceImpl`)**

* **create**: Only admin can add comments. Saves to DB.
* **update**: Updates existing comment text + bitstreamId.
* **delete**: Marks comment as deleted (soft delete).
* **getById**: Fetch single comment.
* **getByBitstreamId**: Fetch all comments for a bitstream (non-deleted).
* **mapToResponse**: Converts `BitstreamComment` entity → `BitstreamCommentResponse` DTO (adds commenter name via `EPersonService`).

---

### 🔹 **Controller (`BitstreamCommentController`)**

Defines REST endpoints:

* Uses `ContextUtil.obtainContext(request)` → get logged-in user context.
* Delegates to service layer.
* Returns `BitstreamCommentResponse` DTOs.

---

## 5. 📖 Line-by-Line Documentation

---

### **Entity: `BitstreamComment`**

```java
@Entity
@Table(name = "bitstream_comment")
public class BitstreamComment {
```

➡ Maps this class to DB table `bitstream_comment`.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private int id;
```

➡ Auto-generated primary key.

```java
@Column(nullable = false)
private String text;
```

➡ The actual comment text.

```java
@Column(name = "bitstream_id", nullable = false)
private UUID bitstreamId;
```

➡ Links comment to a bitstream UUID.

```java
@Column(name = "commenter_id", nullable = false)
private UUID commenterId;
```

➡ Stores which user created the comment.

```java
@Column(name = "comment_date", nullable = false)
private LocalDateTime commentDate;
```

➡ Date/time comment was created.

```java
@Column(name = "is_deleted", nullable = false)
private boolean isDeleted;
```

➡ Soft delete flag.

```java
@PrePersist
protected void onCreate() {
    this.commentDate = LocalDateTime.now();
}
```

➡ Automatically sets creation time before persisting.

---

### **Service Impl – `create()`**

```java
if (!authorizeService.isAdmin(context,user)) {
    throw new RuntimeException("Only admin users are allowed to create comments.");
}
```

➡ Enforces **admin-only comment creation**.

```java
BitstreamComment comment = new BitstreamComment(
        request.getComment(),
        request.getBitstreamId(),
        user.getID()
);
```

➡ Constructs a new `BitstreamComment` entity.

```java
BitstreamComment saved = repository.save(comment);
return mapToResponse(context , saved);
```

➡ Saves to DB, maps to response DTO.

---

### **Service Impl – `delete()`**

```java
comment.setDeleted(true);
repository.save(comment);
```

➡ Marks comment as deleted instead of removing permanently.

---

### **Service Impl – `mapToResponse()`**

```java
EPerson user = ePersonService.find(context, comment.getCommenterId());
String commenterName = (user != null) ? user.getFullName() : "Unknown User";
```

➡ Resolves commenter ID → Full Name.

```java
return new BitstreamCommentResponse(
        comment.getId(),
        comment.getCommentDate(),
        comment.getText(),
        comment.isDeleted(),
        commenterName
);
```

➡ Returns clean response object with `commenterName`.

---

### **Controller – `@GetMapping("/bitstream/{bitstreamId}")`**

```java
public List<BitstreamCommentResponse> getByBitstream(@PathVariable UUID bitstreamId,HttpServletRequest httpRequest) {
    Context context = ContextUtil.obtainContext(httpRequest);
    return service.getByBitstreamId(context,bitstreamId);
}
```

➡ Endpoint to fetch all **non-deleted comments** for a bitstream.

---

# ✅ Summary

This backend provides a **complete comment management system** for bitstreams:

* Stores comments in `bitstream_comment` table.
* Uses **DTOs** for clean request/response.
* Enforces **admin-only creation**.
* Supports **soft delete** to preserve history.
* Exposes **REST APIs** to frontend Angular services.

---
