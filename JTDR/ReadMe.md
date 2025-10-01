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


## 📖 Theory about the above file

* **path: "jtdr"**
  → we are defining the page in which this feature needs to render

* **component: CnrManagerComponent**
  → here we are connecting the component with ts file saying that when i enter this page this component needs to render

* **canActivate: [authenticatedGuard]**
  → here we are restricting only admins can go to this url






**File Location:**  

`dspace_frontend_latest-main/src/app/shared/auth-nav-menu/user-menu/user-menu.component.html`




```ts

<ds-loading *ngIf="(loading$ | async)"></ds-loading>
<ul *ngIf="(loading$ | async) !== true" class="user-menu" role="menu"
  [ngClass]="inExpandableNavbar ? 'user-menu-mobile pb-2 mb-2 border-bottom' : 'user-menu-dropdown'"
  [attr.aria-label]="'nav.user-profile-menu-and-logout' |translate" id="user-menu-dropdown">

  <li class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="['/jtdr']" routerLinkActive="active">Jtdr</a>
  </li>

// other list of buttons and logout button
</ul>

```


## 📖 Theory about the above file

``` ts

incomplete

```

* **role="presentation"**
  → we are checking if the user is admin or not this i boolean if it is true we can see this button

* **component: CnrManagerComponent**
  → here we are connecting the component with ts file saying that when i enter this page this component needs to render

* **canActivate: [authenticatedGuard]**
  → here we are restricting only admins can go to this url








**File Location:**  

`dspace_frontend_latest-main/src/app/shared/auth-nav-menu/user-menu/user-menu.component.ts`




```ts

import {
  AsyncPipe,
  NgClass,
  NgIf,
} from '@angular/common';
import {
  Component,
  EventEmitter,
  Input,
  OnInit,
  Output,
} from '@angular/core';
import {
  RouterLink,
  RouterLinkActive,
} from '@angular/router';
import {
  select,
  Store,
} from '@ngrx/store';
import { TranslateModule } from '@ngx-translate/core';
import { Observable } from 'rxjs';

import { AppState } from '../../../app.reducer';
import {
  getProfileModuleRoute,
  getSubscriptionsModuleRoute,
} from '../../../app-routing-paths';
import { AuthService } from '../../../core/auth/auth.service';
import { isAuthenticationLoading } from '../../../core/auth/selectors';
import { DSONameService } from '../../../core/breadcrumbs/dso-name.service';
import { EPerson } from '../../../core/eperson/models/eperson.model';
import { MYDSPACE_ROUTE } from '../../../my-dspace-page/my-dspace-page.component';
import { ThemedLoadingComponent } from '../../loading/themed-loading.component';
import { LogOutComponent } from '../../log-out/log-out.component';

/**
 * This component represents the user nav menu.
 */
@Component({
  selector: 'ds-base-user-menu',
  templateUrl: './user-menu.component.html',
  styleUrls: ['./user-menu.component.scss'],
  standalone: true,
  imports: [NgIf, ThemedLoadingComponent, RouterLinkActive, NgClass, RouterLink, LogOutComponent, AsyncPipe, TranslateModule],
})
export class UserMenuComponent implements OnInit {

  /**
   * The input flag to show user details in navbar expandable menu
   */
  @Input() inExpandableNavbar = false;

  /**
   * Emits an event when the route changes
   */
  @Output() changedRoute: EventEmitter<any> = new EventEmitter<any>();

  /**
   * True if the authentication is loading.
   * @type {Observable<boolean>}
   */
  public loading$: Observable<boolean>;

  /**
   * The authenticated user.
   * @type {Observable<EPerson>}
   */
  public user$: Observable<EPerson>;

  /**
   * The mydspace page route.
   * @type {string}
   */
  public mydspaceRoute = MYDSPACE_ROUTE;

  /**
   * The profile page route
   */
  public profileRoute = getProfileModuleRoute();

  /**
   * The profile page route
   */
  public subscriptionsRoute = getSubscriptionsModuleRoute();

  constructor(
    protected store: Store<AppState>,
    protected authService: AuthService,
    public dsoNameService: DSONameService,
  ) {
  }

  /**
   * Initialize all instance variables
   */
  ngOnInit(): void {

    // set loading
    this.loading$ = this.store.pipe(select(isAuthenticationLoading));

    // set user
    this.user$ = this.authService.getAuthenticatedUserFromStore();

  }

  /**
   * Emits an event when the menu item is clicked
   */
  onMenuItemClick() {
    this.changedRoute.emit();
  }
}


```


## 📖 Theory about the above file

``` ts
incomplete

```

* **role="presentation"**
  → we are checking if the user is admin or not this i boolean if it is true we can see this button

* **component: CnrManagerComponent**
  → here we are connecting the component with ts file saying that when i enter this page this component needs to render

* **canActivate: [authenticatedGuard]**
  → here we are restricting only admins can go to this url







**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr-manager.component.ts`




```ts



import { Component, OnInit } from '@angular/core';
import { finalize, forkJoin, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { CnrService, FileRecord } from './cnr.service';

type SubmitState = 'idle' | 'submitting' | 'submitted' | 'error';
type CheckState = 'idle' | 'checking' | 'checked' | 'error';

@Component({
  selector: 'app-cnr-manager',
  templateUrl: './cnr-manager.component.html',
  styleUrls: ['./cnr-manager.component.scss']
})
export class CnrManagerComponent implements OnInit {
  searchQuery = '';
  loading = false;
  generating = false;
  submitting = false;

  searchResults: FileRecord[] = [];
  selectedSearchFiles: FileRecord[] = [];
  selectedGeneratedFiles: FileRecord[] = [];
  generatedFiles: Array<FileRecord & {
    selected: boolean;
    status: SubmitState;
    checkStatusState: CheckState;
    userFriendlyPostResponse: string;
    userFriendlyCheckResponse: string;
    postResponse?: string;
  }> = [];


  private readonly POST_CODE_MAP: Record<string, string> = {
    '200': 'Submitted successfully.',
    '201': 'Case created.',
    '202': 'Submission accepted for processing.',
    '401': 'CNR must not be null or empty.',
    '402': 'Zip hash must not be null or empty.',
    '403': 'Invalid Zip File.',
    '404': 'Zip name mismatch.',
    '405': 'ZIP hash mismatch.',
    '406': 'Invalid userId.',
    '407': 'UserId not found in JTDR.',
    '409': 'Duplicate case detected. Case already exists for provided CNR.',
    '500': 'Internal Server Error.'
  };

  private readonly CHECK_CODE_MAP: Record<string, string> = {
    '200': 'Checked successfully.',
    '202': 'Check accepted for processing.',
    '401': 'CNR must not be null or empty.',
    '402': 'Zip hash must not be null or empty.',
    '403': 'Invalid Zip File.',
    '405': 'ZIP hash mismatch.',
    '409': 'Duplicate case detected. Case already exists for provided CNR.',
    '500': 'Internal Server Error.'
  };

  constructor(private cnrService: CnrService) {}

  ngOnInit(): void {
    this.loadGeneratedFiles();
  }

  // ---------- Utils ----------
  private asString(v: any): string {
    if (typeof v === 'string') return v;
    try { return JSON.stringify(v); } catch { return String(v); }
  }

  private parsePostResult(input: any): { code?: string; message?: string; ackId?: string; raw: string } {
    const httpStatus = input?.status ? String(input.status) : undefined;

    // Plain object
    if (input && typeof input === 'object' && !('error' in input)) {
      const code = input.statusCode ? String(input.statusCode) : httpStatus;
      return { code, message: input.message, ackId: input.ackId, raw: this.asString(input) };
    }

    // HttpErrorResponse with JSON
    if (input?.error && typeof input.error === 'object') {
      const code = input.error.statusCode ? String(input.error.statusCode) : httpStatus;
      return { code, message: input.error.message ?? input.message, ackId: input.error.ackId, raw: this.asString(input.error) };
    }

    // Text body
    const text: string =
      typeof input?.error === 'string'
        ? input.error
        : (typeof input === 'string' ? input : input?.message ?? '');

    const codeFromText =
      text.match(/"statusCode"\s*:\s*"?(\d{3})"?/i)?.[1] ||
      text.match(/\b(\d{3})\b/)?.[1] ||
      httpStatus;

    const msgFromText = text.match(/"message"\s*:\s*"([^"]+)"/i)?.[1];

    return { code: codeFromText, message: msgFromText, raw: text || this.asString(input) };
  }

  private mapFriendly(map: Record<string, string>, code?: string, message?: string, raw?: string, fallback = 'Unknown response'): string {
    return (code && map[code]) || message || raw || fallback;
  }

  // ---------- Load Generated Files ----------
  loadGeneratedFiles() {
    this.cnrService.getRecords().subscribe({
      next: (response) => {
        const postErrorMap = this.POST_CODE_MAP;
        const checkErrorMap = this.CHECK_CODE_MAP;

        this.generatedFiles = (response as any[]).map((file) => {
          const postResp: string = file.postResponse || '';
          const checkResp: string = file.getCheckResponse || '';

          const postStatusCode = postResp.match(/^(\d{3})/)?.[1];
          const checkStatusCode = checkResp.match(/^(\d{3})/)?.[1];

          let userFriendlyPostResponse = '';
          if (postResp.includes('Folder to zip not found')) {
            userFriendlyPostResponse = 'Folder to zip not found.';
          } else if (postStatusCode && postErrorMap[postStatusCode]) {
            userFriendlyPostResponse = postErrorMap[postStatusCode];
          } else {
            userFriendlyPostResponse = postResp || 'Not submitted yet';
          }

          const userFriendlyCheckResponse =
            (checkStatusCode && checkErrorMap[checkStatusCode]) || checkResp || 'Not verified yet';

          return {
            ...file,
            selected: false,
            status: 'idle',
            checkStatusState: 'idle',
            userFriendlyPostResponse,
            userFriendlyCheckResponse
          };
        });
      },
      error: (err) => {
        console.error('Error fetching generated files:', err);
      }
    });
  }

  // ---------- Search ----------
  searchItems() {
    if (this.searchQuery.trim() === '') {
      this.searchResults = [];
      return;
    }

    this.loading = true;

    this.cnrService.getSearchResults(this.searchQuery)
      .pipe(finalize(() => (this.loading = false)))
      .subscribe({
        next: (response) => {
          const objects = response._embedded?.searchResult?._embedded?.objects || [];
          this.searchResults = objects.map((obj: any) => {
            const metadata = obj._embedded?.indexableObject?.metadata || {};
            return {
              cino: metadata['dc.cino']?.[0]?.value || 'N/A',
              fileName: obj._embedded?.indexableObject?.name || 'N/A',
              hashValue: 'N/A',
              createdAt: obj._embedded?.indexableObject?.lastModified || 'N/A',
              selected: false,
              ackId: obj._embedded?.indexableObject?.ackId || null,
              itemUUID: obj._embedded?.indexableObject?.uuid || null
            } as FileRecord;
          });
        },
        error: (err) => {
          console.error('Search error:', err);
          this.searchResults = [];
        }
      });
  }

  onSearchInput() {
    if (this.searchQuery.trim() === '') {
      this.searchResults = [];
    }
  }

  onSearchSelectionChange() {
    this.selectedSearchFiles = this.searchResults.filter(file => (file as any).selected);
  }

  onGeneratedSelectionChange() {
    this.selectedGeneratedFiles = this.generatedFiles.filter(file => file.selected);
  }

  // ---------- Generate ----------
  generateFiles() {
    if (this.selectedSearchFiles.length === 0) return;

    this.generating = true;

    const calls = this.selectedSearchFiles.map(item =>
      this.cnrService.generate(item.itemUUID).pipe(
        catchError(err => {
          console.error(`Error generating zip for Item UUID: ${item.itemUUID}`, err);
          return of(null);
        })
      )
    );

    forkJoin(calls).pipe(finalize(() => (this.generating = false))).subscribe();
  }

  // ---------- Submit Single ----------
  submitFile(file: FileRecord & { status?: SubmitState; userFriendlyPostResponse?: string; postResponse?: string; ackId?: string }) {
    if (!file.fileName) return;

    file.status = 'submitting';
    file.userFriendlyPostResponse = '';

    this.cnrService.submitCase(file.fileName).subscribe({
      next: (response) => {
        const { code, message, ackId, raw } = this.parsePostResult(response);
        if (ackId) file.ackId = ackId;

        const isSuccess = code ? code.startsWith('2') : true;
        file.status = isSuccess ? 'submitted' : 'error';

        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
        file.postResponse = raw;
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.status = 'error';
        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
        file.postResponse = raw;
      }
    });
  }

  // ---------- Submit Multiple ----------
  submitAllFiles() {
    if (this.selectedGeneratedFiles.length === 0) return;

    this.submitting = true;

    const calls = this.selectedGeneratedFiles.map((item) => {
      item.status = 'submitting';
      item.userFriendlyPostResponse = '';

      return this.cnrService.submitCase(item.fileName).pipe(
        catchError((err) => {
          const { code, message, raw } = this.parsePostResult(err);
          item.status = 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
          item.postResponse = raw;
          return of(null);
        })
      );
    });

    forkJoin(calls)
      .pipe(finalize(() => (this.submitting = false)))
      .subscribe((responses) => {
        responses?.forEach((resp, idx) => {
          if (!resp) return;
          const item = this.selectedGeneratedFiles[idx];
          const { code, message, ackId, raw } = this.parsePostResult(resp);
          if (ackId) item.ackId = ackId;
          const isSuccess = code ? code.startsWith('2') : true;
          item.status = isSuccess ? 'submitted' : 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
          item.postResponse = raw;
        });
      });
  }

  // ---------- Check Status ----------
  checkStatus(file: FileRecord & { checkStatusState?: CheckState; userFriendlyCheckResponse?: string; ackId?: string }) {
    if (!file.ackId) return;

    file.checkStatusState = 'checking';

    this.cnrService.checkStatus(file.ackId).subscribe({
      next: (response) => {
        const { code, message, raw } = this.parsePostResult(response);
        file.checkStatusState = 'checked';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Checked successfully');
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.checkStatusState = 'error';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Status check failed');
      }
    });
  }

  // ---------- Cart ----------
  addToCart() {
    const selected = this.searchResults.filter((f: any) => f.selected);
    this.selectedSearchFiles.push(...selected);

    // de-dupe by fileName
    const dedup = new Map<string, FileRecord>();
    this.selectedSearchFiles.forEach(f => { if (f.fileName) dedup.set(f.fileName, f); });
    this.selectedSearchFiles = Array.from(dedup.values());
  }
}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation











**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr-manager.component.html`




```html



<div class="container p-4">
  <!-- Search Section -->
  <div class="row mb-4">
    <div class="col-md-6">
      <div class="position-relative">
        <div class="input-group">
          <input [(ngModel)]="searchQuery"
                 (input)="onSearchInput()"
                 class="form-control"
                 placeholder="Search..."
                 [disabled]="loading" />
          <button (click)="searchItems()" class="btn btn-primary">
            <span *ngIf="loading" class="spinner-border spinner-border-sm me-1"></span>
            Search
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Action Buttons -->
  <div class="mb-4">
    <div class="row">
      <!-- Add to Cart -->
      <div class="col-md-4">
        <button class="btn btn-secondary mb-3 w-100"
                (click)="addToCart()"
                [disabled]="selectedSearchFiles.length === 0">
          Add to Cart ({{ selectedSearchFiles.length }})
        </button>
      </div>

      <!-- Generate Files -->
      <div class="col-md-4">
        <button (click)="generateFiles()"
                [disabled]="generating || selectedSearchFiles.length === 0"
                class="btn btn-success mb-3 w-100">
          <span *ngIf="generating" class="spinner-border spinner-border-sm me-1"></span>
          {{ generating ? 'Generating...' : 'Generate Files' }}
        </button>
      </div>

      <!-- Post All Files -->
      <div class="col-md-4">
        <button (click)="submitAllFiles()"
                [disabled]="!selectedGeneratedFiles.length || submitting"
                class="btn btn-primary mb-3 w-100">
          <span *ngIf="submitting" class="spinner-border spinner-border-sm me-1"></span>
          {{ submitting ? 'Submitting...' : 'Post All Files' }}
        </button>
      </div>
    </div>
  </div>

  <!-- Search Results Table -->
  <div class="card" *ngIf="searchResults.length > 0">
    <div class="card-header">
      <h5 class="mb-0">Search Results</h5>
    </div>
    <div class="card-body p-0">
      <table class="table table-striped table-hover mb-0">
        <thead>
          <tr>
            <th>Select</th>
            <th>CINO Number</th>
            <th>File Name</th>
            <th>Created At</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let file of searchResults">
            <td>
              <input type="checkbox" [(ngModel)]="file.selected" (change)="onSearchSelectionChange()" />
            </td>
            <td>{{ file.cino }}</td>
            <td>{{ file.fileName }}</td>
            <td>{{ file.createdAt | date:'short' }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <!-- No search results -->
  <div *ngIf="searchResults.length === 0 && searchQuery" class="text-center text-muted py-4">
    No records found for "{{ searchQuery }}".
  </div>

  <!-- Generated Files Table -->
  <div class="card" *ngIf="generatedFiles.length > 0">
    <div class="card-header">
      <h5 class="mb-0">Generated Files</h5>
    </div>
    <div class="card-body p-0">
      <table class="table table-striped table-hover mb-0">
        <thead>
          <tr>
            <th>Select</th>
            <th>File Name</th>
            <th>Created At</th>
            <th>Zip Status</th>
            <th>No. of Files</th>
            <th>Upload Status</th>
            <th>Verify Status</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let file of generatedFiles">
            <td>
              <input type="checkbox" [(ngModel)]="file.selected" (change)="onGeneratedSelectionChange()" />
            </td>
            <td>{{ file.fileName }}</td>
            <td>{{ file.createdAt | date:'short' }}</td>
            <td>{{ file.zipStatus || 'Not created yet' }}</td>
            <td>{{ file.fileCount || '-' }}</td>
            <td>{{ file.userFriendlyPostResponse || 'Not submitted yet' }}</td>
            <td>{{ file.userFriendlyCheckResponse || 'Not verified yet' }}</td>
            <td>
              <!-- Show Submit if no ackId -->
              <ng-container *ngIf="!file.ackId; else checkStatusButtons" [ngSwitch]="file.status">
                <button *ngSwitchCase="'idle'" (click)="submitFile(file)" class="btn btn-primary btn-sm">
                  Submit
                </button>
                <button *ngSwitchCase="'submitting'" class="btn btn-warning btn-sm"
                        [ngStyle]="{'background-color': '#77BFBF', 'border-color': '#5aa0a0'}"
                        disabled>
                  <span class="spinner-border spinner-border-sm me-1"></span> Submitting...
                </button>
                <button *ngSwitchCase="'submitted'" class="btn btn-success btn-sm" disabled>
                  Submitted
                </button>
                <button *ngSwitchCase="'error'" class="btn btn-danger btn-sm" (click)="submitFile(file)">
                  Retry Submit
                </button>
              </ng-container>

              <!-- Show Check Status if ackId exists -->
              <ng-template #checkStatusButtons>
                <ng-container [ngSwitch]="file.checkStatusState">
                  <button *ngSwitchCase="'idle'" (click)="checkStatus(file)" class="btn btn-info btn-sm">
                    Check Status
                  </button>
                  <button *ngSwitchCase="'checking'" class="btn btn-warning btn-sm"
                          [ngStyle]="{'background-color': '#77BFBF', 'border-color': '#5aa0a0'}"
                          disabled>
                    <span class="spinner-border spinner-border-sm me-1"></span> Checking...
                  </button>
                  <button *ngSwitchCase="'checked'" class="btn btn-success btn-sm" disabled>
                    Checked
                  </button>
                  <button *ngSwitchCase="'error'" class="btn btn-danger btn-sm" (click)="checkStatus(file)">
                    Retry Check
                  </button>
                </ng-container>
              </ng-template>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div *ngIf="generatedFiles.length === 0" class="text-center text-muted py-4">
    No generated files found.
  </div>
</div>


```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation








**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr-manager.component.scss`




```scss

.container {
  max-width: 1200px;
  margin: auto;
}
.table {
  background-color: white;
}

// Custom styles for CNR Manager component

.hover-bg-light:hover {
background-color: #f8f9fa !important;
}

.cursor-pointer {
cursor: pointer;
}

// Search dropdown styling
.position-relative {
.position-absolute {
  border: 1px solid #dee2e6;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  
  .border-bottom:last-child {
    border-bottom: none !important;
  }
}
}

// Tag styling
.badge {
font-size: 0.875rem;
padding: 0.5rem 0.75rem;

.btn-close {
  opacity: 0.7;
  
  &:hover {
    opacity: 1;
  }
}
}

// Table improvements
.table {
th {
  background-color: #f8f9fa;
  border-bottom: 2px solid #dee2e6;
  font-weight: 600;
}

td {
  vertical-align: middle;
}
}

// Button group improvements
.btn-group-sm .btn {
padding: 0.25rem 0.5rem;
font-size: 0.875rem;
}

// Loading states
.spinner-border-sm {
width: 1rem;
height: 1rem;
}

// Card styling
.card {
box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
border: 1px solid rgba(0, 0, 0, 0.125);

.card-header {
  background-color: #f8f9fa;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
}
}

// Status badge improvements
.badge {
&.bg-success {
  background-color: #198754 !important;
}

&.bg-primary {
  background-color: #0d6efd !important;
}

&.bg-danger {
  background-color: #dc3545 !important;
}

&.bg-secondary {
  background-color: #6c757d !important;
}
}

// Responsive improvements
@media (max-width: 768px) {
.input-group {
  flex-direction: column;
  
  .form-select,
  .form-control,
  .btn {
    margin-bottom: 0.5rem;
  }
}

.table-responsive {
  font-size: 0.875rem;
}
}

// Animation for dropdown
.position-absolute {
animation: fadeInDown 0.2s ease-out;
}

@keyframes fadeInDown {
from {
  opacity: 0;
  transform: translateY(-10px);
}
to {
  opacity: 1;
  transform: translateY(0);
}
}


```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation








**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr-manager.component.ts`




```ts

import { Component, OnInit } from '@angular/core';
import { finalize, forkJoin, of } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { CnrService, FileRecord } from './cnr.service';

type SubmitState = 'idle' | 'submitting' | 'submitted' | 'error';
type CheckState = 'idle' | 'checking' | 'checked' | 'error';

@Component({
  selector: 'app-cnr-manager',
  templateUrl: './cnr-manager.component.html',
  styleUrls: ['./cnr-manager.component.scss']
})
export class CnrManagerComponent implements OnInit {
  searchQuery = '';
  loading = false;
  generating = false;
  submitting = false;

  searchResults: FileRecord[] = [];
  selectedSearchFiles: FileRecord[] = [];
  selectedGeneratedFiles: FileRecord[] = [];
  generatedFiles: Array<FileRecord & {
    selected: boolean;
    status: SubmitState;
    checkStatusState: CheckState;
    userFriendlyPostResponse: string;
    userFriendlyCheckResponse: string;
    postResponse?: string;
  }> = [];

  // --- Friendly maps ---
  private readonly POST_CODE_MAP: Record<string, string> = {
    '200': 'Submitted successfully.',
    '201': 'Case created.',
    '202': 'Submission accepted for processing.',
    '401': 'CNR must not be null or empty.',
    '402': 'Zip hash must not be null or empty.',
    '403': 'Invalid Zip File.',
    '404': 'Zip name mismatch.',
    '405': 'ZIP hash mismatch.',
    '406': 'Invalid userId.',
    '407': 'UserId not found in JTDR.',
    '409': 'Duplicate case detected. Case already exists for provided CNR.',
    '500': 'Internal Server Error.'
  };

  private readonly CHECK_CODE_MAP: Record<string, string> = {
    '200': 'Checked successfully.',
    '202': 'Check accepted for processing.',
    '401': 'CNR must not be null or empty.',
    '402': 'Zip hash must not be null or empty.',
    '403': 'Invalid Zip File.',
    '405': 'ZIP hash mismatch.',
    '409': 'Duplicate case detected. Case already exists for provided CNR.',
    '500': 'Internal Server Error.'
  };

  constructor(private cnrService: CnrService) {}

  ngOnInit(): void {
    this.loadGeneratedFiles();
  }

  // ---------- Utils ----------
  private asString(v: any): string {
    if (typeof v === 'string') return v;
    try { return JSON.stringify(v); } catch { return String(v); }
  }

  private parsePostResult(input: any): { code?: string; message?: string; ackId?: string; raw: string } {
    const httpStatus = input?.status ? String(input.status) : undefined;

    // Plain object
    if (input && typeof input === 'object' && !('error' in input)) {
      const code = input.statusCode ? String(input.statusCode) : httpStatus;
      return { code, message: input.message, ackId: input.ackId, raw: this.asString(input) };
    }

    // HttpErrorResponse with JSON
    if (input?.error && typeof input.error === 'object') {
      const code = input.error.statusCode ? String(input.error.statusCode) : httpStatus;
      return { code, message: input.error.message ?? input.message, ackId: input.error.ackId, raw: this.asString(input.error) };
    }

    // Text body
    const text: string =
      typeof input?.error === 'string'
        ? input.error
        : (typeof input === 'string' ? input : input?.message ?? '');

    const codeFromText =
      text.match(/"statusCode"\s*:\s*"?(\d{3})"?/i)?.[1] ||
      text.match(/\b(\d{3})\b/)?.[1] ||
      httpStatus;

    const msgFromText = text.match(/"message"\s*:\s*"([^"]+)"/i)?.[1];

    return { code: codeFromText, message: msgFromText, raw: text || this.asString(input) };
  }

  private mapFriendly(map: Record<string, string>, code?: string, message?: string, raw?: string, fallback = 'Unknown response'): string {
    return (code && map[code]) || message || raw || fallback;
  }

  // ---------- Load Generated Files ----------
  loadGeneratedFiles() {
    this.cnrService.getRecords().subscribe({
      next: (response) => {
        const postErrorMap = this.POST_CODE_MAP;
        const checkErrorMap = this.CHECK_CODE_MAP;

        this.generatedFiles = (response as any[]).map((file) => {
          const postResp: string = file.postResponse || '';
          const checkResp: string = file.getCheckResponse || '';

          const postStatusCode = postResp.match(/^(\d{3})/)?.[1];
          const checkStatusCode = checkResp.match(/^(\d{3})/)?.[1];

          let userFriendlyPostResponse = '';
          if (postResp.includes('Folder to zip not found')) {
            userFriendlyPostResponse = 'Folder to zip not found.';
          } else if (postStatusCode && postErrorMap[postStatusCode]) {
            userFriendlyPostResponse = postErrorMap[postStatusCode];
          } else {
            userFriendlyPostResponse = postResp || 'Not submitted yet';
          }

          const userFriendlyCheckResponse =
            (checkStatusCode && checkErrorMap[checkStatusCode]) || checkResp || 'Not verified yet';

          return {
            ...file,
            selected: false,
            status: 'idle',
            checkStatusState: 'idle',
            userFriendlyPostResponse,
            userFriendlyCheckResponse
          };
        });
      },
      error: (err) => {
        console.error('Error fetching generated files:', err);
      }
    });
  }

  // ---------- Search ----------
  searchItems() {
    if (this.searchQuery.trim() === '') {
      this.searchResults = [];
      return;
    }

    this.loading = true;

    this.cnrService.getSearchResults(this.searchQuery)
      .pipe(finalize(() => (this.loading = false)))
      .subscribe({
        next: (response) => {
          const objects = response._embedded?.searchResult?._embedded?.objects || [];
          this.searchResults = objects.map((obj: any) => {
            const metadata = obj._embedded?.indexableObject?.metadata || {};
            return {
              cino: metadata['dc.cino']?.[0]?.value || 'N/A',
              fileName: obj._embedded?.indexableObject?.name || 'N/A',
              hashValue: 'N/A',
              createdAt: obj._embedded?.indexableObject?.lastModified || 'N/A',
              selected: false,
              ackId: obj._embedded?.indexableObject?.ackId || null,
              itemUUID: obj._embedded?.indexableObject?.uuid || null
            } as FileRecord;
          });
        },
        error: (err) => {
          console.error('Search error:', err);
          this.searchResults = [];
        }
      });
  }

  onSearchInput() {
    if (this.searchQuery.trim() === '') {
      this.searchResults = [];
    }
  }

  onSearchSelectionChange() {
    this.selectedSearchFiles = this.searchResults.filter(file => (file as any).selected);
  }

  onGeneratedSelectionChange() {
    this.selectedGeneratedFiles = this.generatedFiles.filter(file => file.selected);
  }

  // ---------- Generate ----------
  generateFiles() {
    if (this.selectedSearchFiles.length === 0) return;

    this.generating = true;

    const calls = this.selectedSearchFiles.map(item =>
      this.cnrService.generate(item.itemUUID).pipe(
        catchError(err => {
          console.error(`Error generating zip for Item UUID: ${item.itemUUID}`, err);
          return of(null);
        })
      )
    );

    forkJoin(calls).pipe(finalize(() => (this.generating = false))).subscribe();
  }

  // ---------- Submit Single ----------
  submitFile(file: FileRecord & { status?: SubmitState; userFriendlyPostResponse?: string; postResponse?: string; ackId?: string }) {
    if (!file.fileName) return;

    file.status = 'submitting';
    file.userFriendlyPostResponse = '';

    this.cnrService.submitCase(file.fileName).subscribe({
      next: (response) => {
        const { code, message, ackId, raw } = this.parsePostResult(response);
        if (ackId) file.ackId = ackId;

        const isSuccess = code ? code.startsWith('2') : true;
        file.status = isSuccess ? 'submitted' : 'error';

        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
        file.postResponse = raw;
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.status = 'error';
        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
        file.postResponse = raw;
      }
    });
  }

  // ---------- Submit Multiple ----------
  submitAllFiles() {
    if (this.selectedGeneratedFiles.length === 0) return;

    this.submitting = true;

    const calls = this.selectedGeneratedFiles.map((item) => {
      item.status = 'submitting';
      item.userFriendlyPostResponse = '';

      return this.cnrService.submitCase(item.fileName).pipe(
        catchError((err) => {
          const { code, message, raw } = this.parsePostResult(err);
          item.status = 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
          item.postResponse = raw;
          return of(null);
        })
      );
    });

    forkJoin(calls)
      .pipe(finalize(() => (this.submitting = false)))
      .subscribe((responses) => {
        responses?.forEach((resp, idx) => {
          if (!resp) return;
          const item = this.selectedGeneratedFiles[idx];
          const { code, message, ackId, raw } = this.parsePostResult(resp);
          if (ackId) item.ackId = ackId;
          const isSuccess = code ? code.startsWith('2') : true;
          item.status = isSuccess ? 'submitted' : 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
          item.postResponse = raw;
        });
      });
  }

  // ---------- Check Status ----------
  checkStatus(file: FileRecord & { checkStatusState?: CheckState; userFriendlyCheckResponse?: string; ackId?: string }) {
    if (!file.ackId) return;

    file.checkStatusState = 'checking';

    this.cnrService.checkStatus(file.ackId).subscribe({
      next: (response) => {
        const { code, message, raw } = this.parsePostResult(response);
        file.checkStatusState = 'checked';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Checked successfully');
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.checkStatusState = 'error';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Status check failed');
      }
    });
  }

  // ---------- Cart ----------
  addToCart() {
    const selected = this.searchResults.filter((f: any) => f.selected);
    this.selectedSearchFiles.push(...selected);

    // de-dupe by fileName
    const dedup = new Map<string, FileRecord>();
    this.selectedSearchFiles.forEach(f => { if (f.fileName) dedup.set(f.fileName, f); });
    this.selectedSearchFiles = Array.from(dedup.values());
  }
}


```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation











**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr-manager.module.ts`




```ts
// src/app/cnr-manager/cnr-manager.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CnrManagerComponent } from './cnr-manager.component';
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [
    CnrManagerComponent
  ],
  imports: [
    CommonModule,
    FormsModule,
    HttpClientModule
  ],
  exports: [
    CnrManagerComponent
  ]
})
export class CnrManagerModule { }


```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation











**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr.module.ts`




```ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CnrManagerComponent } from './cnr-manager.component';
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [CnrManagerComponent],
  imports: [CommonModule, FormsModule, HttpClientModule],
  exports: [CnrManagerComponent]
})
export class CnrModule {}



```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation



**File Location:**  

`dspace_frontend_latest-main/src/app/cnr-manager/cnr.service.ts`




```ts




import { Injectable } from '@angular/core';
import { HttpClient , HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from 'src/app/core/serachpage/api-urls';

export interface FileRecord {
  fileName: string;
  hashValue: string;
  createdAt: string;
  ackId?: string;
  cino: string; // Add the cino property
  selected: boolean; // For tracking selection
  posted: boolean; // For tracking post status
  itemUUID: string; // Add the itemUUID property
  status?: 'idle' | 'submitting' | 'submitted' | 'error';
  checkStatusState?: 'idle' | 'checking' | 'checked' | 'error'; 
  userFriendlyPostResponse?: string;
  userFriendlyCheckResponse?: string;
  postResponse?: string; // Add this here
}

export interface SearchResult {
  id: string;
  uuid: string;
  name: string;
  handle: string;
  metadata: any;
}

@Injectable({ providedIn: 'root' })
export class CnrService {
  private baseUrl = `${CURRENT_API_URL}/server/api`;

  constructor(private http: HttpClient) {}

  generate(itemUUID: string): Observable<any> {
    return this.http.post<any>(`${this.baseUrl}/export/zip/${itemUUID}`, {});
  }

  getRecords(): Observable<FileRecord[]> {
    return this.http.get<FileRecord[]>(`${this.baseUrl}/cnr/records`);
  }

  submitCase(cnr: string): Observable<any> {
    const params = new HttpParams()
      .set('cnr', cnr);
    return this.http.post(`${this.baseUrl}/jtdr/submit`, null, { params });
  }

  checkStatus(ackId: string): Observable<any> {
    return this.http.get(`${this.baseUrl}/jtdr/status/${ackId}`);
  }

  getSearchResults(query: string): Observable<any> {
    const params = new HttpParams().set('query', query);
    return this.http.get<any>(`${this.baseUrl}/discover/search/objects`, { params });
  }
}  

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation



  

You can follow tis backend integration to implement this feature






**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/FileHashController.java`

```java


package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.app.rest.diracai.service.FileHashService;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.core.Context;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;

@RestController
@RequestMapping("/api/cnr")
public class FileHashController {

    @Autowired
    private FileHashService fileHashService;

    @Autowired
    private FileHashRecordRepository recordRepository;

    // POST: Generate ZIP + SHA256 + Store
    @PostMapping("/generate")
    public FileHashRecord generateAndStoreHash(@RequestParam("cnr") String cnr,
                                               @RequestParam("docType") String docType,
                                               Context context) throws IOException {
        return fileHashService.generateZipAndHash(cnr, context, docType);
    }

    @GetMapping("/records")
    public ResponseEntity<Page<FileHashRecord>> getAllHashes(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "createdAt") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDir,
            @RequestParam(required = false) String submitted
    ) {
        try {
            Sort sort = sortDir.equalsIgnoreCase("asc")
                    ? Sort.by(sortBy).ascending()
                    : Sort.by(sortBy).descending();

            Pageable pageable = PageRequest.of(page, size, sort);
            Page<FileHashRecord> result;

            if ("submit".equalsIgnoreCase(submitted)) {
                result = recordRepository.findByAckIdIsNotNullAndAckIdNot(pageable, "");
            } else if ("notSubmitted".equalsIgnoreCase(submitted)) {
                result = recordRepository.findByAckIdIsNullOrAckId(pageable, "");
            } else {
                result = recordRepository.findAll(pageable);
            }

            return ResponseEntity.ok(result);
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    /** DELETE: remove the generated zip and its DB record by fileName */
    @DeleteMapping("/records/{fileName}")
    public ResponseEntity<Void> deleteGeneratedRecord(@PathVariable String fileName) {
        try {
            FileHashService.DeleteResult result = fileHashService.deleteZipAndRecord(fileName);
            if (result == FileHashService.DeleteResult.NOT_FOUND) {
                return ResponseEntity.notFound().build();
            }
            return ResponseEntity.noContent().build(); // 204
        } catch (IOException ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    /** Optional alias: DELETE /api/cnr/zip/{fileName} */
    @DeleteMapping("/zip/{fileName}")
    public ResponseEntity<Void> deleteGeneratedZip(@PathVariable String fileName) {
        try {
            FileHashService.DeleteResult result = fileHashService.deleteZipAndRecord(fileName);
            if (result == FileHashService.DeleteResult.NOT_FOUND) {
                return ResponseEntity.notFound().build();
            }
            return ResponseEntity.noContent().build();
        } catch (IOException ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation




**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/Repository/FileHashRecordRepository.java`


```java

package org.dspace.app.rest.diracai.Repository;

import org.dspace.content.Diracai.FileHashRecord;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;
import java.util.List;

public interface FileHashRecordRepository extends JpaRepository<FileHashRecord, Long> {
    FileHashRecord findByFileName(String cnr);
    FileHashRecord findByAckId(String ackId);
    List<FileHashRecord> findAll(Sort sort);
    @Query("""
        select f from FileHashRecord f
        where f.createdAt between :from and :to
        order by f.createdAt desc
    """)
    List<FileHashRecord> findAllForReport(@Param("from") LocalDateTime from,
                                          @Param("to") LocalDateTime to);
    Page<FileHashRecord> findByAckIdIsNotNullAndAckIdNot(Pageable pageable, String emptyValue);
    Page<FileHashRecord> findByAckIdIsNullOrAckId(Pageable pageable, String emptyValue);
}

```

## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation



**File Location:**  

`dspace_backend_latest-main/dspace-api/src/main/java/org/dspace/content/Diracai/FileHashRecord.java`

```java

package org.dspace.content.Diracai;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(name = "file_hash_record")
public class FileHashRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String fileName;
    private String hashValue;
    private LocalDateTime createdAt;
    private String ackId;
    private String zipStatus;
    private String postResponse;
    private String postStatus;
    private String getCheckResponse;
    private String getCheckStatus;
    private Integer fileCount;
    private LocalDateTime uploadDate;
    private String batchName;
    private String caseType;
    private String caseNo;
    private String Status;
    private String cinoNumber;
    private String createdBy;
    private String uploadedBy;

    public String getCreatedBy() {
        return createdBy;
    }

    public String getUploadedBy() {
        return uploadedBy;
    }

    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }

    public void setUploadedBy(String uploadedBy) {
        this.uploadedBy = uploadedBy;
    }

    public String getCinoNumber() {
        return cinoNumber;
    }

    public void setCinoNumber(String cinoNumber) {
        this.cinoNumber = cinoNumber;
    }

    public String getStatus() {
        return Status;
    }

    public void setStatus(String status) {
        Status = status;
    }

    public void setUploadDate(LocalDateTime uploadDate) {
        this.uploadDate = uploadDate;
    }

    public void setFileCount(Integer fileCount) {
        this.fileCount = fileCount;
    }

    public LocalDateTime getUploadDate() {
        return uploadDate;
    }

    public void setCaseType(String caseType) {
        this.caseType = caseType;
    }

    public void setBatchName(String batchName) {
        this.batchName = batchName;
    }

    public String getCaseType() {
        return caseType;
    }

    public String getCaseNo() {
        return caseNo;
    }

    public String getBatchName() {
        return batchName;
    }

    public void setCaseNo(String caseNo) {
        this.caseNo = caseNo;
    }

    public Integer getFileCount() {
        return fileCount;
    }

    public void setFileCount(int fileCount) {
        this.fileCount = fileCount;
    }

    @PrePersist
    public void onCreate() {
        this.createdAt =  LocalDateTime.now();
    }


    public String getZipStatus() {
        return zipStatus;
    }

    public String getGetCheckResponse() {
        return getCheckResponse;
    }

    public String getGetCheckStatus() {
        return getCheckStatus;
    }

    public String getPostResponse() {
        return postResponse;
    }

    public String getPostStatus() {
        return postStatus;
    }

    public void setGetCheckResponse(String getCheckResponse) {
        this.getCheckResponse = getCheckResponse;
    }

    public void setGetCheckStatus(String getCheckStatus) {
        this.getCheckStatus = getCheckStatus;
    }

    public void setPostResponse(String postResponse) {
        this.postResponse = postResponse;
    }

    public void setPostStatus(String postStatus) {
        this.postStatus = postStatus;
    }

    public void setZipStatus(String zipStatus) {
        this.zipStatus = zipStatus;
    }

    public FileHashRecord() {}


    // Getters and setters

    public void setAckId(String ackId) {
        this.ackId = ackId;
    }

    public String getAckId() {
        return ackId;
    }

    public Long getId() { return id; }

    public void setId(Long id) { this.id = id; }

    public String getFileName() { return fileName; }

    public void setFileName(String fileName) { this.fileName = fileName; }

    public String getHashValue() { return hashValue; }

    public void setHashValue(String hashValue) { this.hashValue = hashValue; }

    public LocalDateTime getCreatedAt() { return createdAt; }

    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation



**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/JtdrIntegrationController.java`


``` java


package org.dspace.app.rest.diracai.controller;

import jakarta.servlet.http.HttpServletResponse;
import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;

// PDFBox
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.common.PDRectangle;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;

@RestController
@RequestMapping("/api/jtdr")
public class JtdrIntegrationController {

    @Autowired
    private JtdrIntegrationService jtdrService;

    @PostMapping("/submit")
    public Map<String, Object> submitCase(@RequestParam("cnr") String cnr) {
        var context = org.dspace.app.rest.utils.ContextUtil.obtainCurrentRequestContext();
        return jtdrService.submitCase(context, cnr);
    }

    @GetMapping("/status/{ackId}")
    public Map<String, Object> getStatus(@PathVariable String ackId) {
        return jtdrService.checkStatus(ackId);
    }

    /** CSV version (Accept: text/csv) */
    @GetMapping(value = "/report", produces = "text/csv")
    public void reportCsv(
            @RequestParam("start") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
            @RequestParam("end")   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end,
            HttpServletResponse response
    ) throws IOException {
        LocalDateTime from = start.atStartOfDay();
        LocalDateTime to   = end.atTime(23, 59, 59, 999_000_000);
        List<JtdrDetailedReportRow> rows = jtdrService.getDetailedReport(from, to);

        response.setContentType("text/csv");
        response.setHeader("Content-Disposition", "attachment; filename=detailed-report.csv");

        try (PrintWriter writer = response.getWriter()) {
            writer.println("Sl No,Batch Name,Case Type,Case No,Case Year,Zip Created At,Zip Created By,Upload Date,Upload Status,File Submitted By,Zip Status");
            for (JtdrDetailedReportRow row : rows) {
                writer.print(row.slNumber); writer.print(",");
                writer.print(csvSafe(row.batchName)); writer.print(",");
                writer.print(csvSafe(row.caseType)); writer.print(",");
                writer.print(csvSafe(row.caseNo)); writer.print(",");
                writer.print(csvSafe(row.caseYear)); writer.print(",");
                writer.print(row.zipCreatedAt != null ? row.zipCreatedAt : ""); writer.print(",");
                writer.print(csvSafe(row.createdBy)); writer.print(",");
                writer.print(row.uploadDate != null ? row.uploadDate : ""); writer.print(",");
                writer.print(csvSafe(row.uploadStatus)); writer.print(",");
                writer.print(csvSafe(row.uploadedBy)); writer.print(",");
                writer.print(csvSafe(row.zipStatus));
                writer.println();
            }
        }
    }

    /** PDF version (Accept: application/pdf) */
    @GetMapping(value = "/report", produces = "application/pdf")
    public void reportPdf(
            @RequestParam("start") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
            @RequestParam("end")   @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end,
            HttpServletResponse response
    ) throws IOException {
        LocalDateTime from = start.atStartOfDay();
        LocalDateTime to   = end.atTime(23, 59, 59, 999_000_000);
        List<JtdrDetailedReportRow> rows = jtdrService.getDetailedReport(from, to);

        response.setContentType("application/pdf");
        response.setHeader("Content-Disposition", "attachment; filename=detailed-report.pdf");

        try (PDDocument doc = new PDDocument()) {
            PDPage page = new PDPage(PDRectangle.A4);
            doc.addPage(page);

            float margin = 36; // 0.5 inch
            float y = page.getMediaBox().getHeight() - margin;
            float lineHeight = 14f;
            float left = margin;

            PDPageContentStream cs = new PDPageContentStream(doc, page);
            cs.setFont(PDType1Font.HELVETICA_BOLD, 12);
            cs.beginText();
            cs.newLineAtOffset(left, y);
            cs.showText("JTDR Detailed Report");
            cs.endText();

            y -= (lineHeight * 2);

            // Header row
            String[] headers = new String[] {
                    "Sl No","Batch Name","Case Type","Case No","Upload Date",
                    "Upload Status","Zip Created At","Zip Created By","File Submitted By","Zip Status"
            };

            cs.setFont(PDType1Font.HELVETICA_BOLD, 9);
            y = drawWrappedLine(cs, page, left, y, lineHeight, join(headers, " | "));

            cs.setFont(PDType1Font.HELVETICA, 9);
            for (JtdrDetailedReportRow r : rows) {
                String line = String.format(
                        "%s | %s | %s | %s | %s | %s | %s | %s | %s | %s",
                        safe(r.slNumber),
                        safe(r.batchName),
                        safe(r.caseType),
                        safe(r.caseNo),
                        safe(r.uploadDate),
                        safe(r.uploadStatus),
                        safe(r.zipCreatedAt),
                        safe(r.createdBy),
                        safe(r.uploadedBy),
                        safe(r.zipStatus)
                );
                y = drawWrappedLine(cs, page, left, y, lineHeight, line);
                if (y < margin + lineHeight) { // new page
                    cs.close();
                    page = new PDPage(PDRectangle.A4);
                    doc.addPage(page);
                    cs = new PDPageContentStream(doc, page);
                    cs.setFont(PDType1Font.HELVETICA, 9);
                    y = page.getMediaBox().getHeight() - margin;
                }
            }
            cs.close();

            doc.save(response.getOutputStream());
        }
    }

    private static String csvSafe(Object v) {
        if (v == null) return "";
        String s = v.toString();
        if (s.contains(",") || s.contains("\"")) {
            return "\"" + s.replace("\"", "\"\"") + "\"";
        }
        return s;
    }

    private static String safe(Object v) {
        return v == null ? "" : v.toString();
    }

    private static String join(String[] arr, String sep) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < arr.length; i++) {
            if (i > 0) sb.append(sep);
            sb.append(arr[i]);
        }
        return sb.toString();
    }

    /** Simple line writer with wrapping (approximate) */
    private static float drawWrappedLine(PDPageContentStream cs, PDPage page, float x, float y, float lh, String text) throws IOException {
        float maxWidth = page.getMediaBox().getWidth() - (x * 2);
        String[] words = text.split("\\s+");
        StringBuilder line = new StringBuilder();
        for (String w : words) {
            String trial = line.length() == 0 ? w : line + " " + w;
            float width = PDType1Font.HELVETICA.getStringWidth(trial) / 1000 * 9; // 9pt font
            if (width > maxWidth) {
                cs.beginText(); cs.newLineAtOffset(x, y); cs.showText(line.toString()); cs.endText();
                y -= lh;
                line = new StringBuilder(w);
            } else {
                line = new StringBuilder(trial);
            }
        }
        if (line.length() > 0) {
            cs.beginText(); cs.newLineAtOffset(x, y); cs.showText(line.toString()); cs.endText();
            y -= lh;
        }
        return y;
    }
}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation





**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/JtdrIntegrationService.java`



```java

package org.dspace.app.rest.diracai.service;


import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.core.Context;
import org.springframework.web.multipart.MultipartFile;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Map;

public interface JtdrIntegrationService {
    Map<String, Object> submitCase(Context context, String cnr);
    Map<String, Object> checkStatus(String ackId);
    List<JtdrDetailedReportRow> getDetailedReport(LocalDateTime from, LocalDateTime to);
}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation




**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/JtdrIntegrationServiceImpl.java`


```java


package org.dspace.app.rest.diracai.service.impl;

import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.app.rest.diracai.dto.JtdrDetailedReportRow;
import org.dspace.app.rest.diracai.service.JtdrIntegrationService;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.dspace.xoai.services.api.context.ContextService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.FileSystemResource;
import org.springframework.http.*;
import org.springframework.stereotype.Service;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.ssl.SSLContextBuilder;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

import javax.net.ssl.SSLContext;

import java.security.MessageDigest;
import java.nio.file.Files;
import java.nio.file.Path;
import java.math.BigInteger;



import java.io.File;

import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

import static org.dspace.app.rest.diracai.util.InsecureRestTemplateFactory.getInsecureRestTemplate;


@Service
@Slf4j
public class JtdrIntegrationServiceImpl implements JtdrIntegrationService {

    @Autowired
    private FileHashRecordRepository repository;


    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private FileHashRecordRepository recordRepository;

    @Autowired private ItemService itemService;
    @Autowired private ContextService contextService;

    private Map<String, String> responseMessages = new HashMap<>();

    @Override
    public List<JtdrDetailedReportRow> getDetailedReport(LocalDateTime from, LocalDateTime to) {
        var records = repository.findAllForReport(from, to);

        try (Context context = contextService.getContext()) {
            // Capture current user name before stream
            String userName = context.getCurrentUser() != null ? context.getCurrentUser().getName() : "Unknown";

            // Use AtomicInteger for counter inside stream
            AtomicInteger counter = new AtomicInteger(1);

            return records.stream().map(rec -> {
                JtdrDetailedReportRow row = new JtdrDetailedReportRow();
                row.batchName = rec.getBatchName();
                row.caseType = rec.getCaseType();
                row.caseNo = rec.getCaseNo();
                row.slNumber = counter.getAndIncrement();
                row.uploadDate = rec.getUploadDate();
                row.userName = userName;
                row.uploadStatus = rec.getGetCheckResponse();
                row.zipCreatedAt = rec.getCreatedAt();
                row.zipStatus = rec.getZipStatus();
                row.createdBy = rec.getCreatedBy();
                row.uploadedBy = rec.getUploadedBy();
                return row;
            }).collect(Collectors.toList());

        } catch (Exception e) {
            return List.of();
        }
    }


    public String Mappers(String status) {
        // Initialize the response messages map
        responseMessages.put("200", "Case Received Successfully");
        responseMessages.put("401", "CNR must not be null or empty.");
        responseMessages.put("402", "Zip hash must not be null or empty.");
        responseMessages.put("403", "Invalid Zip File");
        responseMessages.put("404", "Invalid Zip: Zip name does not match with CNR");
        responseMessages.put("405", "Provided ZIP hash does not match actual file hash.");
        responseMessages.put("406", "Invalid userId");
        responseMessages.put("407", "Provided userId does not match any user in JTDR application");
        responseMessages.put("409", "Duplicate case detected. Case already exists for provided CNR.");
        responseMessages.put("500", "Internal Server Error");

        return responseMessages.get(status);
    }

    @Override
    public Map<String, Object> submitCase(Context context,String cnr) {
        try {

            String dspaceDir = configurationService.getProperty("dspace.dir");
            String folderBasePath = dspaceDir + "/jtdr";
            FileHashRecord fileHashRecord = recordRepository.findByFileName(cnr);
            String cino = fileHashRecord.getCinoNumber();
            String zipFilePath = folderBasePath + "/" + cino + ".zip";
            File zipFile = new File(zipFilePath);


            generateZip(cino, zipFile);

            if (!zipFile.exists()) {
                return Map.of("error", "ZIP file not found", "path", zipFilePath);
            }

            String url = "https://orissa.jtdr.gov.in/api/add/case";

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.MULTIPART_FORM_DATA);


            MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
            String calculatedZipHash = calculateSHA256(zipFile);
            body.add("zipHash", calculatedZipHash);
            body.add("cnr", cino);
            body.add("caseZip", new FileSystemResource(zipFile));
            body.add("userId", "depositor_hc@orissa.hc.in");

            HttpEntity<MultiValueMap<String, Object>> request = new HttpEntity<>(body, headers);
            RestTemplate restTemplate = getInsecureRestTemplate();

            ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);

            String responseText = response.getBody();

            // Extract status code and message from the response text
            String statusCode = extractStatusCode(responseText);


            if (responseMap.containsKey("ackId")) {
                FileHashRecord record = repository.findByFileName(cnr);
                if (record != null) {
                    record.setUploadDate(LocalDateTime.now());
                    record.setAckId((String) responseMap.get("ackId"));
                    record.setPostResponse((String) responseMap.getOrDefault("message", ""));
                    record.setStatus("Transferred Case Successfully");
                    record.setUploadedBy(context.getCurrentUser().getName());
                    repository.save(record);
                }
            } else {
                FileHashRecord record = repository.findByFileName(cnr);
                if (record != null) {
                    record.setUploadDate(LocalDateTime.now());
                    record.setPostStatus(statusCode);
                    record.setPostResponse(Mappers(statusCode));
                    repository.save(record);
                }
            }


            return responseMap;

        } catch (Exception e) {
            FileHashRecord record = repository.findByFileName(cnr);
            if (record != null) {
                record.setPostResponse(e.getMessage());
                repository.save(record);
            }
            return Map.of("error", "Failed to submit case", "details", e.getMessage());
        }
    }

    // Extracts the status code from the response text (e.g., "409")
    private String extractStatusCode(String responseText) {
        String[] parts = responseText.split(" ");
        return parts[0];
    }


    @Override
    public Map<String, Object> checkStatus(String ackId) {
        try {
            String url = "https://orissa.jtdr.gov.in/api/status/case?ackId=" + ackId;

            RestTemplate restTemplate = getInsecureRestTemplate();
            ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
            Map<String, Object> responseMap = new ObjectMapper().readValue(response.getBody(), Map.class);

            if (responseMap.containsKey("message")) {
                FileHashRecord record = repository.findByAckId(ackId);
                if (record != null) {
                    record.setGetCheckResponse((String) responseMap.get("message"));
                    repository.save(record);
                }
            }

            return responseMap;

        } catch (Exception e) {
            return Map.of("error", "Failed to get check response", "details", e.getMessage());
        }
    }



    private void generateZip(String cnr, File zipFile) throws IOException {
        String cleanCnr = cnr.replaceAll("\\.zip$", ""); // Remove .zip if present

        File folderToZip = new File(zipFile.getParent(), cleanCnr); // e.g., /home/dspace/dspace/jtdr/ODHC010004612666
        if (!folderToZip.exists() || !folderToZip.isDirectory()) {
            throw new IOException("Folder to zip not found: " + folderToZip.getAbsolutePath());
        }

        try (ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(zipFile))) {
            File[] files = folderToZip.listFiles();
            if (files == null) throw new IOException("Failed to list files in folder: " + folderToZip);

            for (File file : files) {
                String zipEntryName = cleanCnr + "/" + file.getName(); // ✅ Keep consistent folder name inside zip
                zos.putNextEntry(new ZipEntry(zipEntryName));
                Files.copy(file.toPath(), zos);
                zos.closeEntry();
            }
        }
    }

    private String calculateSHA256(File file) throws Exception {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] fileBytes = Files.readAllBytes(file.toPath());
        byte[] hashBytes = digest.digest(fileBytes);
        BigInteger number = new BigInteger(1, hashBytes);
        StringBuilder hexString = new StringBuilder(number.toString(16));

        while (hexString.length() < 64) {
            hexString.insert(0, '0');
        }
        return hexString.toString();
    }


}

```


## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation




**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/dto/JtdrDetailedReportRow.java`

```java

package org.dspace.app.rest.diracai.dto;


import java.time.LocalDateTime;


public class JtdrDetailedReportRow {
    public Integer slNumber;
    public String batchName;
    public String caseType;
    public String caseNo;
    public String zipStatus;
    public LocalDateTime zipCreatedAt;
    public String zipCreatedBy;
    public LocalDateTime uploadDate;
    public String uploadStatus;
    public String userName;
    public String createdBy;
    public String uploadedBy;
    public String caseYear;


}

```


**File Location:**  


`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/controller/ZipExportRestController.java`



``` java

package org.dspace.app.rest.diracai.controller;

import org.dspace.app.rest.diracai.service.ZipExportService;
import org.dspace.app.rest.utils.ContextUtil;
import org.dspace.content.Item;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.File;
import java.util.UUID;

@RestController
@RequestMapping("/api/export")
public class ZipExportRestController {

    @Autowired
    private ZipExportService zipExportService;

    @Autowired
    private ItemService itemService;

    @PostMapping("/zip/{itemUUID}")
    public ResponseEntity<String> generateZip(@PathVariable UUID itemUUID) {
        Context context = null;
        try {
            context = ContextUtil.obtainCurrentRequestContext();
            Item item = itemService.find(context, itemUUID);
            if (item == null) {
                return ResponseEntity.status(HttpStatus.NOT_FOUND).body("Item not found");
            }

            File zip = zipExportService.generateZipForItem(context, item);
            return ResponseEntity.ok("Zip generated: " + zip.getAbsolutePath());

        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error: " + e.getMessage());
        } finally {
            if (context != null) {
                context.abort();
            }
        }
    }
}

```

## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation





**File Location:**  

`dspace_backend_latest-main/dspace-server-webapp/src/main/java/org/dspace/app/rest/diracai/service/ZipExportService.java`



``` java

package org.dspace.app.rest.diracai.service;

import com.amazonaws.util.IOUtils;
import com.fasterxml.jackson.databind.ObjectMapper;
import in.cdac.hcdc.jtdr.metadata.JTDRMetadataSchema;
import in.cdac.hcdc.jtdr.metadata.schema.*;
import org.dspace.app.rest.diracai.Repository.FileHashRecordRepository;
import org.dspace.content.Bitstream;
import org.dspace.content.Bundle;
import org.dspace.content.Diracai.FileHashRecord;
import org.dspace.content.Item;
import org.dspace.content.MetadataValue;
import org.dspace.content.service.BitstreamService;
import org.dspace.content.service.ItemService;
import org.dspace.core.Context;
import org.dspace.services.ConfigurationService;
import org.dspace.storage.bitstore.service.BitstreamStorageService;
import java.time.LocalDateTime;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

@Service
public class ZipExportService {

    @Autowired
    private ItemService itemService;

    @Autowired
    private BitstreamService bitstreamService;

    @Autowired
    private BitstreamStorageService bitstreamStorageService;

    @Autowired
    private ConfigurationService configurationService;

    @Autowired
    private FileHashRecordRepository fileHashRecordRepository;

    public File generateZipForItem(Context context, Item item) throws Exception {
        String cino = getCinoFromMetadata(item,"dc","cino",null);
        String caseType = getCinoFromMetadata(item,"dc","casetype",null);
        String caseNumber = getCinoFromMetadata(item,"dc","case","number");
        String petitionerName = getCinoFromMetadata(item,"dc","pname",null);
        String respondentName = getCinoFromMetadata(item,"dc","rname",null);
        String advocateName = getCinoFromMetadata(item,"dc","paname",null);
        String judgeName = getCinoFromMetadata(item,"dc","contributor","author");
        String disposalDate = getCinoFromMetadata(item,"dc","date","disposal");
        String district = getCinoFromMetadata(item,"dc","district",null);
        String caseYear = getCinoFromMetadata(item,"dc","caseyear",null);
        String scanDate = getCinoFromMetadata(item,"dc","date","scan");
        String verifiedBy = getCinoFromMetadata(item,"dc","verified-by",null);
        String dateVerification = getCinoFromMetadata(item,"dc","date","verification");
        String batchNumber = getCinoFromMetadata(item,"dc","batch-number",null);
        String barcodeNumber = getCinoFromMetadata(item,"dc","barcode",null);
        String fileSize = getCinoFromMetadata(item,"dc","size",null);
        String characterCount = getCinoFromMetadata(item,"dc","char-count",null);
        String noOfPages = getCinoFromMetadata(item,"dc","pages",null);
        String title = getCinoFromMetadata(item,"dc","title",null);
        String docType = getCinoFromMetadata(item,"dc","doc","type");


        if (cino == null) throw new Exception("CINO not found in metadata");

        String dspaceDir = configurationService.getProperty("dspace.dir");

        File baseDir = new File(dspaceDir, "jtdr");
        if (!baseDir.exists()) baseDir.mkdirs();

        File cinoDir = new File(baseDir, cino);
        if (!cinoDir.exists()) cinoDir.mkdirs();

        List<Map<String, String>> docList = new ArrayList<>();
        for (Bundle bundle : item.getBundles("ORIGINAL")) {
            for (Bitstream bitstream : bundle.getBitstreams()) {
                InputStream is = bitstreamService.retrieve(context, bitstream);
                File file = new File(cinoDir, bitstream.getName());
                try (FileOutputStream fos = new FileOutputStream(file)) {
                    IOUtils.copy(is, fos);
                }
                docList.add(Map.of(
                        "docName", bitstream.getName(),
                        "docType", docType,
                        "docDate", LocalDate.now().format(DateTimeFormatter.ofPattern("dd-MM-yyyy"))
                ));
            }

        }

        // Step 3: Write <CINO>_doc.json
        File jsonFile = new File(cinoDir, cino + "_doc.json");
        new ObjectMapper().writerWithDefaultPrettyPrinter()
                .writeValue(jsonFile, docList);

        // Step 4: Write <CINO>_Metadata.xml
        File xmlFile = new File(cinoDir, cino + "_Metadata.xml");
        generateMetadataXml(item, cino ,caseType, caseNumber,petitionerName,respondentName,advocateName, judgeName ,disposalDate,district, caseYear,scanDate,verifiedBy,dateVerification,barcodeNumber,fileSize,characterCount,noOfPages,title,docType,xmlFile);


        // Step 5: Create zip file
        File zipFile = new File(baseDir, cino + ".zip");
        try (FileOutputStream fos = new FileOutputStream(zipFile);
             ZipOutputStream zos = new ZipOutputStream(fos)) {
            zipFolder(cinoDir, cinoDir.getName(), zos);
        }

        FileHashRecord fileHashRecord = new FileHashRecord();
        fileHashRecord.setFileName(caseType+"_"+title+"_"+caseYear);
        fileHashRecord.setBatchName(batchNumber);
        fileHashRecord.setCaseType(caseType);
        fileHashRecord.setCaseNo(caseNumber);
        fileHashRecord.setStatus("Zip File Created");
        fileHashRecord.setCreatedAt(LocalDateTime.now());
        fileHashRecord.setCinoNumber(cino);
        fileHashRecord.setFileCount(docList.size());
        fileHashRecord.setCreatedBy(context.getCurrentUser().getName());
        fileHashRecordRepository.save(fileHashRecord);
        return zipFile;
    }


    private String getCinoFromMetadata(Item item,String schema, String element , String qualifier) {
        List<MetadataValue> values = itemService.getMetadata(item, schema, element, qualifier, Item.ANY);
        return values.isEmpty() ? null : values.get(0).getValue();
    }

    private void generateMetadataXml(
            Item item,
            String cino,
            String caseType,
            String caseNumber,
            String petitionerName,
            String respondentName,
            String advocateName,
            String judgeName,
            String disposalDate,
            String district,
            String caseYear,
            String scanDate,
            String verifiedBy,
            String dateVerification,
            String barcodeNumber,
            String fileSize,
            String characterCount,
            String noOfPages,
            String title,
            String docType,
            File xmlOutputFile
    )

    {
        ECourtCaseType eCourtCaseType = new ECourtCaseType();

        System.out.println(
                "cino=" + cino +
                        ", caseType=" + caseType +
                        ", caseNumber=" + caseNumber +
                        ", petitionerName=" + petitionerName +
                        ", respondentName=" + respondentName +
                        ", advocateName=" + advocateName +
                        ", judgeName=" + judgeName +
                        ", disposalDate=" + disposalDate +
                        ", district=" + district +
                        ", caseYear=" + caseYear +
                        ", scanDate=" + scanDate +
                        ", verifiedBy=" + verifiedBy +
                        ", dateVerification=" + dateVerification +
                        ", barcodeNumber=" + barcodeNumber +
                        ", fileSize=" + fileSize +
                        ", characterCount=" + characterCount +
                        ", noOfPages=" + noOfPages +
                        ", title=" + title
        );

        CaseTypeInformation caseTypeInformation = new CaseTypeInformation();
        caseTypeInformation.setCaseCNRNumber(cino);
        caseTypeInformation.setCaseNature(caseType);
        caseTypeInformation.setCaseNumber(title);
        caseTypeInformation.setCaseTypeName(caseType);
        caseTypeInformation.setNameOfDistrict(district);
        caseTypeInformation.setRegistrationYear(caseYear);
        caseTypeInformation.setRegistrationDate(caseYear+"-01-01 00:00:00");
        caseTypeInformation.setRegistrationNumber(title);
//        caseTypeInformation.setDocType(docType);

        eCourtCaseType.setCase(caseTypeInformation);

        LitigantTypeInformation litigant = new LitigantTypeInformation();
        PetitionerTypeInformation petitioner = new PetitionerTypeInformation();
        petitioner.setPetitionerName(petitionerName);
        litigant.getPetitioner().add(petitioner);


        RespondentTypeInformation respondent = new RespondentTypeInformation();
        respondent.setRespondentName(respondentName);
        litigant.getRespondent().add(respondent);

        caseTypeInformation.setLitigant(litigant);

        JudgeTypeInformation judgeTypeInformation = new JudgeTypeInformation();

        JudgeInformation judgeInformation = new JudgeInformation();
        judgeInformation.setJudgeName(judgeName);
        judgeTypeInformation.getJudgeInfo().add(judgeInformation);
        eCourtCaseType.setJudge(judgeTypeInformation);


        StatusOfCasesTypeInformation statusOfCasesTypeInformation = new StatusOfCasesTypeInformation();
        statusOfCasesTypeInformation.setDateOfDisposal(disposalDate);
        eCourtCaseType.setStatusOfCases(statusOfCasesTypeInformation);


        DigitizationTypeInformation digitizationTypeInformation = new DigitizationTypeInformation();
        MasterFileType masterFileType = new MasterFileType();
        masterFileType.setFileSize(fileSize);
        digitizationTypeInformation.setMasterFile(masterFileType);
        digitizationTypeInformation.setVerifiedBy(verifiedBy);
        eCourtCaseType.setDigitization(digitizationTypeInformation);





        AdvocateTypeInformation advocate = new AdvocateTypeInformation();
        advocate.setAdvocateName(advocateName);
        caseTypeInformation.getAdvocate().add(advocate);


        JTDRMetadataSchema.createXML(eCourtCaseType, xmlOutputFile.getAbsolutePath());
    }



    private void zipFolder(File folder, String basePath, ZipOutputStream zos) throws Exception {
        for (File file : folder.listFiles()) {
            if (file.isDirectory()) {
                zipFolder(file, basePath + "/" + file.getName(), zos);
            } else {
                try (FileInputStream fis = new FileInputStream(file)) {
                    ZipEntry entry = new ZipEntry(basePath + "/" + file.getName());
                    zos.putNextEntry(entry);
                    IOUtils.copy(fis, zos);
                    zos.closeEntry();
                }
            }
        }
    }
}


```

## 📖 Theory about the above file

``` ts
which method we are discussing

```

* **lineContent**
  → line by line explanation

