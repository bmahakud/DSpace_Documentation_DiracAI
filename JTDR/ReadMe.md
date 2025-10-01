# 🏛️ JTDR Module

## Overview
The **JTDR (Judicial Data Transmission & Repository)** module allows administrators to generate, manage, and upload case files in a standardized **ZIP format** to the JTDR server.  

This module ensures proper creation of court-related case bundles, integrity validation using **SHA-256 hashing**, and seamless status tracking of uploaded files.

---

## 🔑 Key Features

### 1. Admin Access
- Only **admins** can see the **JTDR navigation button**.  
- Clicking the button navigates to the **JTDR management page**.

---

### 2. Search & Filters
- Two ways to search for case items:
  1. **By CINO** (Case Identification Number)  
  2. **By Case Details** – Case Type, Case Number, Case Year  

- Based on the selected option, relevant **search filters** appear.  
- Clicking **Search** shows a **list of case items**.

---

### 3. Case Item List
- Each search result shows:
  - **CINO**  
  - **File Name** (Format: `<caseType_caseNumber_caseYear>`)  
  - **Action** – *Add to List*  

- Clicking **Add to List** moves the item into the **View List**.

---

### 4. View List & File Generation
- Clicking **View List** opens a **popup** with all selected files.  
- User can generate a **ZIP file** from the selected items.  

**Backend Workflow:**
1. A folder with the **CINO number** is created in `root/jtdr/`.  
2. A **ZIP file** containing all case documents is generated.  
3. File information (CINO, file name, path) is stored in the **database**.  

---

### 5. File Integrity & Upload
- Before sending files to the JTDR server:
  - A **SHA-256 hash** is generated for the ZIP file.  
  - The system prepares an API request with:
    - **File Name**
    - **User Name**
    - **Hash Code**

- Each file entry in the UI shows:
  - File details  
  - Action buttons:
    - **Upload** – Sends file to JTDR server  
    - **Check Status** – Verifies if upload was successful  

---

### 6. Upload Results
- ✅ **Success:** File shows *“Uploaded Successfully”*.  
- ❌ **Failure:** Error message returned by JTDR API is displayed.  
- 🔄 **Status Check:** Allows tracking of file upload status from JTDR server.

---

## 📂 File Structure Example

```

root/jtdr/
└── <CINO_NUMBER>/
└── <CINO_NUMBER>.zip

```



You can follow this frontend integration to implement this feature


**File Location:**  

`dspace_frontend_latest-main/src/app/app-routes.ts`

```ts

import type { InMemoryScrollingOptions, Route, RouterConfigOptions } from "@angular/router"

import { CnrManagerComponent } from "./cnr-manager/cnr-manager.component"
import { AdminPannelComponent } from "./admin-pannel/admin-pannel.component"

// other neccessary imports

export const APP_ROUTES: Route[] = [
  { path: INTERNAL_SERVER_ERROR, component: ThemedPageInternalServerErrorComponent },
  { path: ERROR_PAGE, component: ThemedPageErrorComponent },
  {
    path: "",
    canActivate: [authBlockingGuard],
    canActivateChild: [ServerCheckGuard],
    resolve: [menuResolver],
    children: [
      { path: "", redirectTo: "/home", pathMatch: "full" },

      // add other paths
      {
        path: "jtdr",
        component: CnrManagerComponent,
        canActivate: [authenticatedGuard],
      },
      // add other paths 
    ],
  },
]
export const APP_ROUTING_CONF: RouterConfigOptions = {
  onSameUrlNavigation: "reload",
}
export const APP_ROUTING_SCROLL_CONF: InMemoryScrollingOptions = {
  scrollPositionRestoration: "top",
  anchorScrolling: "enabled",
}

```

**Theory about the above file:**

```ts

      {
        path: "jtdr",
        component: CnrManagerComponent,
        canActivate: [authenticatedGuard],
      },

```

path:"jtdr" --> we are defining the page in which this feature needs to render
component: CnrManagerComponent --> here we are connecting the component with ts file saying that when i enter this page this component needs to render
canActivate: [authenticatedGuard] --> here we are restricting only admins can go to this url 



## 📖 Theory about the above file

* **path: "jtdr"**
  → we are defining the page in which this feature needs to render

* **component: CnrManagerComponent**
  → here we are connecting the component with ts file saying that when i enter this page this component needs to render

* **canActivate: [authenticatedGuard]**
  → here we are restricting only admins can go to this url




You can follow tis backend integration to implement this feature

