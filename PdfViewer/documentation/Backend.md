
# 📄 API Documentation: `BitstreamPermissionController`

---

## 1. 📂 File Location

This class should be placed in:

```
[dspace-src]/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/BitstreamPermissionController.java
```

---

## 2. ✅ Features of This API

* **Endpoint**: `GET /api/custom/bitstreams/{uuid}/permissions`
* **Purpose**:

  * Fetch **current user’s permissions** (print, download, access time window) for a given bitstream.
  * Check if the user is an **admin**.
  * If the file is **PDF** and past its **disposal date**, auto-convert to **PDF/A**, replace content in DSpace, and record status.
* **Response**: JSON containing:

  ```json
  {
    "bitstreamId": "UUID",
    "userId": 123,
    "isAdmin": true,
    "pdfaConverted": false,
    "policies": [
      {
        "name": "Policy Name",
        "description": "Some description",
        "policyType": "TYPE",
        "action": "READ",
        "startDate": "2025-09-20 10:00:00",
        "endDate": "2025-09-20 18:00:00",
        "pageStart": 1,
        "pageEnd": 10,
        "print": true,
        "download": false
      }
    ]
  }
  ```

---

## 3. 🔍 Feature-wise Explanation

* **User authentication** → Checks current logged-in user (`EPerson`).
* **Admin check** → Uses `AuthorizeService.isAdmin`.
* **Policy check** → Loads `ResourcePolicy` for this bitstream, filters by:

  * Assigned to current user/group.
  * Active (between startDate and endDate).
* **PDF/A conversion**:

  * If `bitstream` is a PDF **AND** item has metadata `dc.date.disposal` **AND** current date > disposal date → convert to PDF/A.
  * Replace original file with converted one.
* **Response JSON**: Returns user permissions, admin flag, and whether PDF/A conversion occurred.

---

## 4. 📖 Line-by-Line Explanation

---

```java
@RestController
@RequestMapping("/api/custom/bitstreams")
public class BitstreamPermissionController {
```

➡ Declares this as a REST controller.
➡ All endpoints start with `/api/custom/bitstreams`.

---

```java
@Autowired
private BitstreamService bitstreamService;
@Autowired
private ResourcePolicyService resourcePolicyService;
@Autowired
private RequestService requestService;
@Autowired
private AuthorizeService authorizeService;
@Autowired
private GroupService groupService;
@Autowired
private ItemService itemService;
```

➡ Injects required DSpace services:

* `BitstreamService`: Find, retrieve, update files.
* `ResourcePolicyService`: Fetch policies for access control.
* `AuthorizeService`: Check admin rights.
* `GroupService`: Check group membership (Anonymous, custom groups).
* `ItemService`: Access item metadata (like disposal date).

---

```java
@GetMapping("/{uuid}/permissions")
public ResponseEntity<?> getUserBitstreamPermissions(@PathVariable UUID uuid,
                                                     HttpServletRequest request) throws Exception {
```

➡ Defines API `GET /api/custom/bitstreams/{uuid}/permissions`.
➡ `uuid` = bitstream identifier.
➡ `request` = used to extract DSpace context (user session).

---

```java
Context context = ContextUtil.obtainContext(request);
context.turnOffAuthorisationSystem();
```

➡ Obtain DSpace context (wraps DB + user session).
➡ Temporarily disables authorization checks → needed to read/update bitstreams.

---

```java
Bitstream bitstream = bitstreamService.findByIdOrLegacyId(context, uuid.toString());
SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
```

➡ Fetch bitstream by UUID.
➡ Prepare date formatter for output.

---

```java
if (bitstream == null) {
    return ResponseEntity.notFound().build();
}
```

➡ If bitstream doesn’t exist → return 404.

---

```java
EPerson currentUser = context.getCurrentUser();
if (currentUser == null) {
    return ResponseEntity.status(401).body("Unauthorized");
}
```

➡ Require user login.
➡ If not logged in → return 401 Unauthorized.

---

```java
String mimeType = bitstream.getFormat(context).getMIMEType();
boolean isPdf = "application/pdf".equalsIgnoreCase(mimeType);
```

➡ Detect if file is a **PDF**.

---

```java
Item owningItem = bitstream.getBundles().stream()
        .flatMap(bundle -> bundle.getItems().stream())
        .findFirst()
        .orElse(null);
```

➡ Find the **item** that owns this bitstream.
➡ Needed to fetch metadata like `dc.date.disposal`.

---

```java
final String METADATA_SCHEMA = "dc";
final String METADATA_ELEMENT = "date";
final String METADATA_QUALIFIER = "disposal";
```

➡ Define metadata field we care about: `dc.date.disposal`.

---

```java
List<MetadataValue> metadataValues = itemService.getMetadata(
        owningItem, METADATA_SCHEMA, METADATA_ELEMENT, METADATA_QUALIFIER, Item.ANY);
```

➡ Fetch disposal date metadata from owning item.

---

```java
Date disposalDate = null;
if (metadataValues != null && !metadataValues.isEmpty()) {
    try {
        disposalDate = new SimpleDateFormat("yyyy-MM-dd").parse(metadataValues.get(0).getValue());
    } catch (Exception e) {
        return ResponseEntity.status(400).body("Invalid disposal date format");
    }
}
```

➡ Parse disposal date from metadata.
➡ If format invalid → return 400 Bad Request.

---

```java
if (isPdf && disposalDate != null && now.after(disposalDate)) {
    try {
        ...
        pdfaFile = PdfaConverter.convertToPdfA(inputFile);
        ...
        replaceBitstreamContent(context, bitstream, pdfaFile);
        pdfaConverted = true;
    } catch (Exception e) {
        return ResponseEntity.status(500).body("Error during PDF/A conversion: " + e.getMessage());
    } finally {
        ...
        context.restoreAuthSystemState();
    }
}
```

➡ If file is PDF and disposal date passed:

1. Retrieve bitstream content.
2. Convert to PDF/A using `PdfaConverter`.
3. Save copy for audit (`/home/dspace/dspace/July_7th/test/`).
4. Replace original content with new PDF/A.
5. Cleanup temp files.
   ➡ If error → return 500.

---

```java
boolean isAdmin = authorizeService.isAdmin(context, currentUser);
```

➡ Check if user is a repository admin.

---

```java
List<ResourcePolicy> allPolicies = resourcePolicyService.find(context, bitstream);
```

➡ Fetch all policies attached to this bitstream.

---

```java
List<Map<String, Object>> userPolicies = allPolicies.stream()
        .filter(policy -> {
            try {
                return ((policy.getEPerson() != null && policy.getEPerson().equals(currentUser)) ||
                        (policy.getGroup() != null &&
                                ("Anonymous".equals(policy.getGroup().getName()) ||
                                        groupService.isMember(context, currentUser, policy.getGroup()))))
                        &&
                        (policy.getStartDate() == null || !now.before(policy.getStartDate()))
                        &&
                        (policy.getEndDate() == null || !now.after(policy.getEndDate()));
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        })
        .map(policy -> {
            Map<String, Object> map = new HashMap<>();
            map.put("name", policy.getRpName());
            map.put("description", policy.getRpDescription());
            map.put("policyType", policy.getRpType());
            map.put("action", resolveActionName(policy.getAction()));
            map.put("startDate", policy.getStartDate() != null ? formatter.format(policy.getStartDate()) : null);
            map.put("endDate", policy.getEndDate() != null ? formatter.format(policy.getEndDate()) : null);
            map.put("pageStart", policy.getPageStart());
            map.put("pageEnd", policy.getPageEnd());
            map.put("print", policy.isPrint());
            map.put("download", policy.isDownload());
            return map;
        })
        .collect(Collectors.toList());
```

➡ Filters policies:

* Belongs to this user OR a group they belong to (Anonymous included).
* Within valid time window.
  ➡ Maps policies into simplified JSON structure with fields like `print`, `download`.

---

```java
return ResponseEntity.ok(Map.of(
        "bitstreamId", uuid.toString(),
        "userId", currentUser.getID(),
        "isAdmin", isAdmin,
        "pdfaConverted", pdfaConverted,
        "policies", userPolicies
));
```

➡ Final response JSON includes:

* Bitstream ID
* User ID
* Admin flag
* Whether PDF/A conversion happened
* List of active policies

---

```java
private void replaceBitstreamContent(Context context, Bitstream bitstream, File newFile) throws Exception {
    try (InputStream newStream = new FileInputStream(newFile)) {
        bitstreamService.updateContents(context, bitstream, newStream);
        bitstream.setSizeBytes(newFile.length());
        bitstreamService.setFormat(context, bitstream, bitstream.getFormat(context));
        bitstreamService.update(context, bitstream);
        context.commit();
    }
}
```

➡ Helper method to replace bitstream content with PDF/A file.
➡ Updates size + format + commits transaction.

---

# ✅ Summary

This API does **three jobs**:

1. 🔐 Checks user login, admin status, and fetches policies.
2. 📜 Filters and formats permissions into JSON for frontend.
3. 📄 If file is PDF & past disposal date → converts to PDF/A & replaces bitstream.
