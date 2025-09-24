# Admin-Pool Feature

The **Admin-Pool** module manages **batch file uploads** and enforces an **approval workflow** with strict role-based access.

---

## 🔑 Key Features

- When a **new batch** is uploaded, its details appear in the **pool**.
- Users with correct permissions can **approve** or **reject** batch files.
- Approved/rejected files move to their **respective pools**.
- Each batch file shows details such as:
  - **Uploader**
  - **Associated Collection**
- Clicking **Perform Task** under a batch file shows **detailed item info**.
- **Role-based views:**
  - **Admin:** View all, approve/reject any.
  - **Reviewer:** View only claimed batch files.
  - **Uploader:** Upload batch files only.

---

## 👥 Role Scopes

| Role     | Scope of Actions                  |
|----------|-----------------------------------|
| Admin    | Upload and review any batch file  |
| Uploader | Upload batch files only           |
| Reviewer | Review batch files only           |

---

## 📂 Frontend Integration

**File:**  
`dspace_frontend_latest-main/src/app/app-routes.ts`

```ts
import type { InMemoryScrollingOptions, Route, RouterConfigOptions } from "@angular/router"

import {
  ERROR_PAGE,
  INTERNAL_SERVER_ERROR,
} from "./app-routing-paths"
import { authBlockingGuard } from "./core/auth/auth-blocking.guard"
import { authenticatedGuard } from "./core/auth/authenticated.guard"
import { endUserAgreementCurrentUserGuard } from "./core/end-user-agreement/end-user-agreement-current-user.guard"
import { ServerCheckGuard } from "./core/server-check/server-check.guard"
import { menuResolver } from "./menuResolver"
import { provideSuggestionNotificationsState } from "./notifications/provide-suggestion-notifications-state"
import { ThemedPageErrorComponent } from "./page-error/themed-page-error.component"
import { ThemedPageInternalServerErrorComponent } from "./page-internal-server-error/themed-page-internal-server-error.component"

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
      {
        path: "adminpool",
        loadChildren: () => import("./admin-pool/admin-pool.module").then((m) => m.AdminPoolModule),
        data: { enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      }
    ]
  },
]
export const APP_ROUTING_CONF: RouterConfigOptions = {
  onSameUrlNavigation: "reload",
}
export const APP_ROUTING_SCROLL_CONF: InMemoryScrollingOptions = {
  scrollPositionRestoration: "top",
  anchorScrolling: "enabled",
}
