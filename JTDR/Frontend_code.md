Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/cnr-manager 
name of the file : cnr-manager-component.html



<div class="container p-4">
  <!-- Search Section -->
  <div class="row mb-4">
    <div class="col-md-12">
      <div class="card">
        <div class="card-body">

          <!-- Mode selector -->
          <div class="row g-2 align-items-end mb-3">
            <div class="col-md-3">
              <label class="form-label">Search Mode</label>
              <select class="form-select" [(ngModel)]="searchMode">
                <option value="cino">By CINO Number</option>
                <option value="case">By Case (Number / Type / Year)</option>
              </select>
            </div>
          </div>

          <!-- CINO Search -->
          <div *ngIf="searchMode === 'cino'" class="row g-2 align-items-end">
            <div class="col-md-6">
              <label class="form-label">CINO Number</label>
              <div class="input-group">
                <input [(ngModel)]="searchQuery" (input)="onSearchInput()" class="form-control"
                  placeholder="Enter CINO number…" [disabled]="loading" />
                <button (click)="searchItems()" class="btn btn-primary">
                  <span *ngIf="loading" class="spinner-border spinner-border-sm me-1"></span>
                  Search
                </button>
              </div>
            </div>
          </div>

          <!-- Case Search -->
          <div *ngIf="searchMode === 'case'" class="row g-2 align-items-end">
            <div class="col-md-3">
              <label class="form-label">Case Number</label>
              <input class="form-control" [(ngModel)]="caseNumberInput" placeholder="e.g., 1234" />
            </div>
            <div class="col-md-3">
              <label class="form-label">Case Type</label>
              <input class="form-control" [(ngModel)]="caseTypeInput" placeholder="e.g., WP(C), CRLMC" />
            </div>
            <div class="col-md-3">
              <label class="form-label">Case Year</label>
              <input class="form-control" [(ngModel)]="caseYearInput" placeholder="e.g., 2024" />
            </div>
            <div class="col-md-3 d-flex gap-2">
              <button class="btn btn-primary w-100" (click)="searchItems()"
                [disabled]="loading || !(caseNumberInput || caseTypeInput || caseYearInput)">
                <span *ngIf="loading" class="spinner-border spinner-border-sm me-1"></span>
                Search
              </button>
              <button class="btn btn-outline-secondary" (click)="clearCaseInputs()" [disabled]="loading">
                Clear
              </button>
            </div>
          </div>

        </div>
      </div>

      <!-- Report card -->
      <div class="card mt-3">
        <div class="card-header">
          <h5 class="mb-0">Report</h5>
        </div>
        <div class="card-body">
          <div class="row g-2 align-items-end">
            <div class="col-md-3">
              <label class="form-label">From</label>
              <input type="date" class="form-control" [(ngModel)]="reportFrom" [max]="reportTo || ''">
            </div>
            <div class="col-md-3">
              <label class="form-label">To</label>
              <input type="date" class="form-control" [(ngModel)]="reportTo" [min]="reportFrom || ''">
            </div>

            <!-- Format selector -->
            <div class="col-md-3">
              <label class="form-label">Format</label>
              <select class="form-select" [(ngModel)]="reportFormat">
                <option value="csv">CSV</option>
                <option value="pdf">PDF</option>
              </select>
            </div>

            <div class="col-md-3">
              <button class="btn btn-outline-dark w-100" (click)="downloadReport()"
                [disabled]="reportLoading || !reportFrom || !reportTo">
                <span *ngIf="reportLoading" class="spinner-border spinner-border-sm me-2"></span>
                {{ reportLoading ? 'Generating…' : 'Download Report' }}
              </button>
            </div>
          </div>
          <small class="text-muted d-block mt-2">
            Tip: Dates are inclusive. Report contains metadata, zip status, submission & verification info.
          </small>
        </div>
      </div>

    </div>
  </div>

  <!-- Top Actions: View Cart + Generate + Post -->
  <div class="mb-4">
    <div class="row">
      <div class="col-md-4">
        <button class="btn btn-outline-dark mb-3 w-100" (click)="openCart()">
          Add to Zip List ({{ cartCount }})
        </button>
      </div>

      <div class="col-md-4">
        <button (click)="submitAllFiles()" [disabled]="!selectedGeneratedFiles.length || submitting"
          class="btn btn-primary mb-3 w-100">
          <span *ngIf="submitting" class="spinner-border spinner-border-sm me-1"></span>
          {{ submitting ? 'Submitting...' : 'Post All Files' }}
        </button>
      </div>
    </div>
  </div>

  <!-- Search Results -->
  <div class="card" *ngIf="searchResults.length > 0">
    <div class="card-header">
      <h5 class="mb-0">Search Results</h5>
    </div>
    <div class="card-body p-0">
      <table class="table table-striped table-hover mb-0">
        <thead>
          <tr>
            <th>CINO Number</th>
            <th>File Name</th>
            <th>Created At</th>
            <th style="width: 160px">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let file of searchResults">
            <td>{{ file.cino }}</td>
            <td>{{ file.fileName }}</td>
            <td>{{ file.createdAt | date: 'short' }}</td>
            <td>
              <button
                class="btn btn-sm"
                [ngClass]="isInCart(file) ? 'btn-secondary' : 'btn-outline-primary'"
                (click)="addSingleToCart(file)"
                [disabled]="isInCart(file)"
              >
                {{ isInCart(file) ? 'Added' : 'Add to Zip List' }}
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <!-- No results message -->
  <div
    *ngIf="
      searchResults.length === 0 &&
      ((searchMode === 'cino' && searchQuery) ||
       (searchMode === 'case' && (caseNumberInput || caseTypeInput || caseYearInput)))
    "
    class="text-center text-muted py-4"
  >
    No records found.
  </div>

  <!-- Generated Files -->
  <div class="card" *ngIf="generatedFiles.length > 0">
    <div class="card-header d-flex justify-content-between align-items-center">
      <h5 class="mb-0">Generated Files</h5>

      <!-- Filters + Sort -->
      <div class="d-flex align-items-center">
        <div class="me-3">
          <label class="me-1">Filter:</label>
          <select [(ngModel)]="filterSubmitted">
            <option value="">All</option>
            <option value="submit">Submitted</option>
            <option value="notSubmitted">Not Submitted</option>
          </select>
        </div>

        <div class="me-3">
          <label class="me-1">Sort:</label>
          <select [(ngModel)]="sortDir">
            <option value="desc">DESC</option>
            <option value="asc">ASC</option>
          </select>
        </div>

        <button class="btn btn-sm btn-primary" (click)="applyFilters()">
          Apply
        </button>
      </div>
    </div>

    <!-- Table -->
    <div class="card-body p-0">
      <table class="table table-striped table-hover mb-0">
        <thead>
          <tr>
            <th>Select</th>
            <th>File Name</th>
            <th>CINO Number</th>
            <th>Created At</th>
            <th>No. of Files</th>
            <th>File Processing</th>
            <th>Delivery Status</th>
            <th>Actions</th>
          </tr>          
        </thead>
        <tbody>
          <tr *ngFor="let file of generatedFiles">
            <td>
              <input type="checkbox" [(ngModel)]="file.selected" (change)="onGeneratedSelectionChange()" />
            </td>
            <td>{{ file.fileName }}</td>
            <td>{{ file.cinoNumber }}</td>
            <td>{{ file.createdAt | date:'short' }}</td>
            <td>{{ file.fileCount || '-' }}</td>
            <td>{{ file.userFriendlyPostResponse }}</td>
            <td>{{ file.userFriendlyCheckResponse || 'Not verified yet' }}</td>
            <td>
              <!-- Post / Check block -->
              <ng-container *ngIf="!file.ackId; else checkStatusButtons" [ngSwitch]="file.status">
                <button *ngSwitchCase="'idle'" (click)="submitFile(file)" class="btn btn-primary btn-sm">
                  Submit
                </button>
                <button *ngSwitchCase="'submitting'" class="btn btn-warning btn-sm"
                  [ngStyle]="{'background-color': '#77BFBF', 'border-color': '#5aa0a0'}" disabled>
                  <span class="spinner-border spinner-border-sm me-1"></span> Submitting...
                </button>
                <button *ngSwitchCase="'submitted'" class="btn btn-success btn-sm" disabled>
                  Submitted
                </button>
                <button *ngSwitchCase="'error'" class="btn btn-danger btn-sm" (click)="submitFile(file)">
                  Retry Submit
                </button>
              </ng-container>

              <ng-template #checkStatusButtons>
                <ng-container [ngSwitch]="file.checkStatusState">
                  <button *ngSwitchCase="'idle'" (click)="checkStatus(file)" class="btn btn-info btn-sm">
                    Check Status
                  </button>
                  <button *ngSwitchCase="'checking'" class="btn btn-warning btn-sm"
                    [ngStyle]="{'background-color': '#77BFBF', 'border-color': '#5aa0a0'}" disabled>
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

              <!-- Delete generated zip -->
              <button class="btn btn-outline-danger btn-sm ms-2"
                      (click)="deleteGenerated(file)"
                      [disabled]="file.deleting">
                <span *ngIf="file.deleting" class="spinner-border spinner-border-sm me-1"></span>
                Delete
              </button>

            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="card-footer d-flex justify-content-center">
      <button class="btn btn-sm btn-outline-primary me-2" (click)="loadGeneratedFiles(currentPage - 1)"
        [disabled]="currentPage === 0">Previous</button>
      Page {{ currentPage + 1 }} / {{ totalPages }}
      <button class="btn btn-sm btn-outline-primary ms-2" (click)="loadGeneratedFiles(currentPage + 1)"
        [disabled]="currentPage + 1 >= totalPages">Next</button>
    </div>
  </div>

  <div *ngIf="generatedFiles.length === 0" class="text-center text-muted py-4">
    No generated files found.
  </div>
</div>

<!-- CART MODAL -->
<div *ngIf="showCart">
  <div class="modal-backdrop fade show" (click)="closeCart()"></div>

  <div class="modal d-block" tabindex="-1" role="dialog" aria-modal="true">
    <div class="modal-dialog modal-lg modal-dialog-centered">
      <div class="modal-content" (click)="$event.stopPropagation()">
        <div class="modal-header">
          <h5 class="modal-title">Cart ({{ cartCount }})</h5>
          <button type="button" class="btn-close" aria-label="Close" (click)="closeCart()"></button>
        </div>

        <div class="modal-body p-0">
          <ng-container *ngIf="cartCount; else emptyCart">
            <div class="table-responsive">
              <table class="table table-striped table-hover mb-0">
                <thead>
                  <tr>
                    <th style="width: 40%">File Name</th>
                    <th style="width: 25%">CINO</th>
                    <th style="width: 25%">Created At</th>
                    <th style="width: 10%">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr *ngFor="let row of cartView; trackBy: trackByCart">
                    <td>{{ row.fileName }}</td>
                    <td>{{ row.cino }}</td>
                    <td>
                      <span *ngIf="row.createdAt; else dash">
                        {{ row.createdAt | date:'short' }}
                      </span>
                      <ng-template #dash>—</ng-template>
                    </td>
                    <td>
                      <button class="btn btn-sm btn-outline-danger"
                        (click)="removeFromCart(row._raw)">
                        Remove
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </ng-container>

          <ng-template #emptyCart>
            <div class="text-center text-muted py-5">
              Your cart is empty.
            </div>
          </ng-template>
        </div>

        <div class="modal-footer d-flex justify-content-between">
          <button class="btn btn-outline-danger" (click)="clearCart()" [disabled]="!cartCount">
            Clear Cart
          </button>
          <div>
            <button class="btn btn-secondary me-2" (click)="closeCart()">Close</button>
            <button class="btn btn-success"
              (click)="closeCart(); generateFiles()"
              [disabled]="!cartCount || generating">
              <span *ngIf="generating" class="spinner-border spinner-border-sm me-1"></span>
              {{ generating ? 'Generating…' : 'Generate Files' }}
            </button>
          </div>
        </div>

      </div>
    </div>
  </div>
</div>
<!-- /CART MODAL -->












Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/cnr-manager 
name of the file : cnr-manager-component.scss



.container {
  max-width: 1200px;
  margin: auto;
}
.table {
  background-color: white;
}


.hover-bg-light:hover {
background-color: #f8f9fa !important;
}

.cursor-pointer {
cursor: pointer;
}

.position-relative {
.position-absolute {
  border: 1px solid #dee2e6;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  
  .border-bottom:last-child {
    border-bottom: none !important;
  }
}
}

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

.btn-group-sm .btn {
padding: 0.25rem 0.5rem;
font-size: 0.875rem;
}

.spinner-border-sm {
width: 1rem;
height: 1rem;
}

.card {
box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
border: 1px solid rgba(0, 0, 0, 0.125);

.card-header {
  background-color: #f8f9fa;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
}
}

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



Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/cnr-manager 
name of the file : cnr-manager-component.ts



import { Component, OnInit, ChangeDetectorRef } from '@angular/core';
import { finalize, forkJoin, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators';
import { CnrService, FileRecord } from './cnr.service';
import { SearchPageService } from '../core/serachpage/search-page.service';

type SubmitState = 'idle' | 'submitting' | 'submitted' | 'error';
type CheckState = 'idle' | 'checking' | 'checked' | 'error';

@Component({
  selector: 'app-cnr-manager',
  templateUrl: './cnr-manager.component.html',
  styleUrls: ['./cnr-manager.component.scss']
})
export class CnrManagerComponent implements OnInit {
  searchMode: 'cino' | 'case' = 'cino';
  searchQuery = '';
  caseNumberInput = '';
  caseTypeInput = '';
  caseYearInput = '';

  loading = false;
  generating = false;
  submitting = false;

  currentPage = 0;
  pageSize = 10;
  totalPages = 0;
  filterSubmitted: '' | 'submit' | 'notSubmitted' = '';
  sortDir: 'asc' | 'desc' = 'desc';

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
    ackId?: string | null;
    deleting?: boolean; // <-- NEW: row-level deleting flag
  }> = [];

  reportFrom: string | null = null;
  reportTo: string | null = null;
  reportLoading = false;
  reportFormat: 'csv' | 'pdf' = 'csv'; // <-- NEW: report format selector

  private cartKeySet = new Set<string>();
  cartItems: FileRecord[] = [];

  showCart = false;

  constructor(
    private cnrService: CnrService,
    private cdr: ChangeDetectorRef,
    private searchPageService: SearchPageService
  ) {}

  ngOnInit(): void {
    this.loadGeneratedFiles();
  }

  private fileKey(f: FileRecord): string {
    return (f as any).itemUUID || f.fileName || '';
  }

  get cartCount(): number {
    return this.cartItems.length;
  }

  isInCart(file: FileRecord): boolean {
    const key = this.fileKey(file);
    return !!key && this.cartKeySet.has(key);
  }

  addSingleToCart(file: FileRecord): void {
    const key = this.fileKey(file);
    if (!key || this.cartKeySet.has(key)) return;
    this.cartKeySet.add(key);
    this.cartItems = [...this.cartItems, file];
    this.cdr.detectChanges();
  }

  removeFromCart(file: FileRecord): void {
    const key = this.fileKey(file);
    if (!key) return;
    if (this.cartKeySet.delete(key)) {
      this.cartItems = this.cartItems.filter(f => this.fileKey(f) !== key);
      this.cdr.detectChanges();
    }
  }

  clearCart(): void {
    this.cartKeySet.clear();
    this.cartItems = [];
    this.cdr.detectChanges();
  }

  get cartView() {
    return this.cartItems.map(it => ({
      cino: (it as any).cino ?? (it as any).cinoNumber ?? 'N/A',
      fileName: it.fileName ?? 'N/A',
      createdAt: (it as any).createdAt ?? null,
      _raw: it
    }));
  }

  trackByCart(index: number, row: any) {
    const r = row?._raw ?? row;
    return r?.itemUUID ?? r?.fileName ?? index;
  }

  openCart(): void { this.showCart = true; this.cdr.detectChanges(); }
  closeCart(): void { this.showCart = false; this.cdr.detectChanges(); }
  viewCart(): void { this.openCart(); }

  downloadReportCsv() {
    this.reportFormat = 'csv';
    this.downloadReport();
  }

  downloadReport() {
    if (!this.reportFrom || !this.reportTo) return;
    if (new Date(this.reportFrom) > new Date(this.reportTo)) {
      alert('From date cannot be after To date');
      return;
    }

    this.reportLoading = true;
    this.cdr.detectChanges();

    const handleBlob = (blob: Blob, ext: 'csv' | 'pdf') => {
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `jtdr_report_${this.reportFrom}_to_${this.reportTo}.${ext}`;
      document.body.appendChild(a);
      a.click();
      a.remove();
      window.URL.revokeObjectURL(url);
      this.reportLoading = false;
      this.cdr.detectChanges();
    };

    if (this.reportFormat === 'csv') {
      this.cnrService.getReportCsv(this.reportFrom, this.reportTo).subscribe({
        next: (blob: Blob) => handleBlob(blob, 'csv'),
        error: (err) => {
          console.error('Report CSV failed:', err);
          this.reportLoading = false;
          alert('Failed to generate report CSV.');
          this.cdr.detectChanges();
        }
      });
    } else {
      // PDF branch
      this.cnrService.getReportPdf(this.reportFrom, this.reportTo).subscribe({
        next: (blob: Blob) => handleBlob(blob, 'pdf'),
        error: (err) => {
          console.error('Report PDF failed:', err);
          this.reportLoading = false;
          alert('Failed to generate report PDF.');
          this.cdr.detectChanges();
        }
      });
    }
  }

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

  private asString(v: any): string {
    if (typeof v === 'string') return v;
    try { return JSON.stringify(v); } catch { return String(v); }
  }

  private parsePostResult(input: any): { code?: string; message?: string; ackId?: string; raw: string } {
    const httpStatus = input?.status ? String(input.status) : undefined;

    if (input && typeof input === 'object' && !('error' in input)) {
      const code = input.statusCode ? String(input.statusCode) : httpStatus;
      return { code, message: input.message, ackId: input.ackId, raw: this.asString(input) };
    }

    if (input?.error && typeof input.error === 'object') {
      const code = input.error.statusCode ? String(input.error.statusCode) : httpStatus;
      return { code, message: input.error.message ?? input.message, ackId: input.error.ackId, raw: this.asString(input.error) };
    }

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

  private mdFirst(md: any, primary: string, alts: string[] = []): string | undefined {
    const keys = [primary, ...alts];
    for (const k of keys) {
      const arr = md?.[k];
      if (Array.isArray(arr) && arr[0]?.value != null) {
        return String(arr[0].value);
      }
    }
    return undefined;
  }

  private buildDisplayFileName(md: any): string {
    const title    = this.mdFirst(md, 'dc.title');
    const caseType = this.mdFirst(md, 'dc.casetype', ['dc.case.type']);
    const caseYear = this.mdFirst(md, 'dc.caseyear', ['dc.case.year']);
    const parts = [title, caseType, caseYear].filter(Boolean);
    return parts.length ? parts.join('_') : 'N/A';
  }

  loadGeneratedFiles(page: number = this.currentPage) {
    this.cnrService.getRecords(
      page,
      this.pageSize,
      this.filterSubmitted === '' ? undefined : this.filterSubmitted,
      'createdAt',
      this.sortDir
    )
    .subscribe({
      next: (response: any) => {
        this.totalPages  = response.totalPages ?? 0;
        this.currentPage = response.number ?? 0;

        const postErrorMap  = this.POST_CODE_MAP;
        const checkErrorMap = this.CHECK_CODE_MAP;
        const records = Array.isArray(response.content) ? response.content : [];

        this.generatedFiles = records.map((file: any) => {
          const postResp: string  = file.postResponse ?? '';
          const checkResp: string = file.getCheckResponse ?? '';

          const postStatusCode: string | undefined =
            (file.postStatus != null ? String(file.postStatus) : undefined) ||
            postResp.match(/^(\d{3})/)?.[1];

          const checkStatusCode: string | undefined =
            (file.getCheckStatus != null ? String(file.getCheckStatus) : undefined) ||
            checkResp.match(/^(\d{3})/)?.[1];

          let userFriendlyPostResponse = '';
          if (file.status && String(file.status).trim().length > 0) {
            userFriendlyPostResponse = String(file.status).trim();
          } else if (postResp.includes('Folder to zip not found')) {
            userFriendlyPostResponse = 'Folder to zip not found.';
          } else if (postStatusCode && postErrorMap[postStatusCode]) {
            userFriendlyPostResponse = postErrorMap[postStatusCode];
          } else {
            userFriendlyPostResponse = postResp || 'Not submitted yet';
          }

          const userFriendlyCheckResponse =
            (checkStatusCode && checkErrorMap[checkStatusCode]) ||
            checkResp ||
            'Not verified yet';

          return {
            ...file,
            selected: false,
            status: 'idle',
            checkStatusState: 'idle',
            userFriendlyPostResponse,
            userFriendlyCheckResponse,
            deleting: false
          };
        });

        this.cdr.detectChanges();
      },
      error: (err) => {
        console.error('Error fetching generated files:', err);
        this.generatedFiles = [];
        this.cdr.detectChanges();
      }
    });
  }

  searchItems() {
    this.loading = true;
    this.cdr.detectChanges();

    if (this.searchMode === 'cino') {
      if (this.searchQuery.trim() === '') {
        this.loading = false;
        this.searchResults = [];
        this.cdr.detectChanges();
        return;
      }

      this.cnrService.getSearchResults(this.searchQuery)
        .pipe(finalize(() => {
          this.loading = false;
          this.cdr.detectChanges();
        }))
        .subscribe({
          next: (response) => {
            const objects = response._embedded?.searchResult?._embedded?.objects || [];
            this.searchResults = objects.map((obj: any) => {
              const metadata = obj._embedded?.indexableObject?.metadata || {};
              return {
                cino: metadata['dc.cino']?.[0]?.value || 'N/A',
                fileName: this.buildDisplayFileName(metadata),
                hashValue: 'N/A',
                createdAt: obj._embedded?.indexableObject?.lastModified || 'N/A',
                selected: false,
                ackId: obj._embedded?.indexableObject?.ackId || null,
                itemUUID: obj._embedded?.indexableObject?.uuid || null
              } as FileRecord;
            });
            this.cdr.detectChanges();
          },
          error: (err) => {
            console.error('Search error:', err);
            this.searchResults = [];
            this.cdr.detectChanges();
          }
        });

    } else {
      const caseNumber = this.caseNumberInput?.trim() || undefined;
      const caseType   = this.caseTypeInput?.trim()   || undefined;
      const caseYear   = this.caseYearInput?.trim()   || undefined;

      if (!caseNumber && !caseType && !caseYear) {
        this.loading = false;
        this.cdr.detectChanges();
        return;
      }

      this.searchPageService
        .getSearchResults(caseNumber, caseType, caseYear, 'dc.title', 'ASC', 10)
        .pipe(finalize(() => {
          this.loading = false;
          this.cdr.detectChanges();
        }))
        .subscribe({
          next: (response) => {
            const objects = response?._embedded?.searchResult?._embedded?.objects || [];
            this.searchResults = objects.map((obj: any) => {
              const md = obj._embedded?.indexableObject?.metadata || {};
              return {
                cino: md['dc.cino']?.[0]?.value || 'N/A',
                fileName: this.buildDisplayFileName(md),
                hashValue: 'N/A',
                createdAt: obj._embedded?.indexableObject?.lastModified || 'N/A',
                selected: false,
                ackId: obj._embedded?.indexableObject?.ackId || null,
                itemUUID: obj._embedded?.indexableObject?.uuid || null
              } as FileRecord;
            });
            this.cdr.detectChanges();
          },
          error: (err) => {
            console.error('Case search error:', err);
            this.searchResults = [];
            this.cdr.detectChanges();
          }
        });
    }
  }

  onSearchInput() {
    if (this.searchMode !== 'cino') return;
    if (this.searchQuery.trim() === '') {
      this.searchResults = [];
      this.cdr.detectChanges();
    }
  }

  clearCaseInputs() {
    this.caseNumberInput = '';
    this.caseTypeInput = '';
    this.caseYearInput = '';
    this.cdr.detectChanges();
  }

  onSearchSelectionChange() {
    this.selectedSearchFiles = this.searchResults.filter(file => (file as any).selected);
    this.cdr.detectChanges();
  }
  onGeneratedSelectionChange() {
    this.selectedGeneratedFiles = this.generatedFiles.filter(file => file.selected);
    this.cdr.detectChanges();
  }

  generateFiles() {
    if (this.cartItems.length === 0) return;

    this.generating = true;
    this.cdr.detectChanges();

    const calls = this.cartItems.map(item =>
      this.cnrService.generate(item.itemUUID).pipe(
        catchError(err => {
          console.error(`Error generating zip for Item UUID: ${item.itemUUID}`, err);
          return of(null);
        })
      )
    );

    forkJoin(calls)
      .pipe(finalize(() => {
        this.generating = false;
        // Refresh generated files list without full page reload
        this.loadGeneratedFiles(this.currentPage);
        this.cdr.detectChanges();
      }))
      .subscribe();
  }

  submitFile(file: FileRecord & {
    status?: SubmitState;
    userFriendlyPostResponse?: string;
    postResponse?: string;
    ackId?: string;
    checkStatusState?: CheckState;
    userFriendlyCheckResponse?: string;
  }) {
    if (!file.fileName) return;

    if (file.ackId) {
      this.checkStatus(file as any);
      return;
    }

    file.status = 'submitting';
    file.userFriendlyPostResponse = '';
    this.cdr.detectChanges();

    this.cnrService.submitCase(file.fileName).subscribe({
      next: (response) => {
        const { code, message, ackId, raw } = this.parsePostResult(response);
        if (ackId) (file as any).ackId = ackId;

        const isSuccess = code ? code.startsWith('2') : true;
        file.status = isSuccess ? 'submitted' : 'error';

        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
        file.postResponse = raw;
        this.cdr.detectChanges();
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.status = 'error';
        file.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
        file.postResponse = raw;
        this.cdr.detectChanges();
      }
    });
  }

  submitAllFiles() {
    if (this.selectedGeneratedFiles.length === 0) return;

    this.submitting = true;
    this.cdr.detectChanges();

    const toPost  = this.selectedGeneratedFiles.filter(f => !f.ackId);
    const toCheck = this.selectedGeneratedFiles.filter(f => !!f.ackId);

    const postCalls = toPost.map(item => {
      item.status = 'submitting';
      item.userFriendlyPostResponse = '';
      return this.cnrService.submitCase(item.fileName).pipe(
        map(resp => {
          const { code, message, ackId, raw } = this.parsePostResult(resp);
          if (ackId) item.ackId = ackId;
          const isSuccess = code ? code.startsWith('2') : true;
          item.status = isSuccess ? 'submitted' : 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Unknown response');
          item.postResponse = raw;
          return true;
        }),
        catchError(err => {
          const { code, message, raw } = this.parsePostResult(err);
          item.status = 'error';
          item.userFriendlyPostResponse = this.mapFriendly(this.POST_CODE_MAP, code, message, raw, 'Submission failed');
          item.postResponse = raw;
          return of(false);
        })
      );
    });

    const checkCalls = toCheck.map(item => {
      item.checkStatusState = 'checking';
      return this.cnrService.checkStatus(item.ackId as string).pipe(
        map(resp => {
          const { code, message, raw } = this.parsePostResult(resp);
          item.checkStatusState = 'checked';
          item.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Checked successfully');
          return true;
        }),
        catchError(err => {
          const { code, message, raw } = this.parsePostResult(err);
          item.checkStatusState = 'error';
          item.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Status check failed');
          return of(false);
        })
      );
    });

    const allCalls = [...postCalls, ...checkCalls];

    forkJoin(allCalls.length ? allCalls : [of(true)])
      .pipe(finalize(() => {
        this.submitting = false;
        this.loadGeneratedFiles(this.currentPage); // Refresh list to reflect ackIds/statuses
        this.cdr.detectChanges();
      }))
      .subscribe();
  }

  deleteGenerated(file: FileRecord & { fileName?: string }) {
    if (!file?.fileName) return;
    const row = this.generatedFiles.find(f => f.fileName === file.fileName);
    if (row) row.deleting = true;
    this.cdr.detectChanges();

    this.cnrService.deleteGenerated(file.fileName)
      .pipe(finalize(() => {
        if (row) row.deleting = false;
        this.cdr.detectChanges();
      }))
      .subscribe({
        next: () => {
          // Refresh list after successful delete
          this.loadGeneratedFiles(this.currentPage);
        },
        error: (err) => {
          console.error('Delete failed:', err);
          alert('Failed to delete file.');
        }
      });
  }

  checkStatus(file: FileRecord & { checkStatusState?: CheckState; userFriendlyCheckResponse?: string; ackId?: string }) {
    if (!file.ackId) return;

    file.checkStatusState = 'checking';
    this.cdr.detectChanges();

    this.cnrService.checkStatus(file.ackId).subscribe({
      next: (response) => {
        const { code, message, raw } = this.parsePostResult(response);
        file.checkStatusState = 'checked';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Checked successfully');
        this.cdr.detectChanges();
      },
      error: (err) => {
        const { code, message, raw } = this.parsePostResult(err);
        file.checkStatusState = 'error';
        file.userFriendlyCheckResponse = this.mapFriendly(this.CHECK_CODE_MAP, code, message, raw, 'Status check failed');
        this.cdr.detectChanges();
      }
    });
  }

  applyFilters() {
    this.currentPage = 0;
    this.loadGeneratedFiles(0);
  }

}






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



// src/app/cnr-manager/cnr.module.ts
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


import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from 'src/app/core/serachpage/api-urls';

export interface FileRecord {
  fileName: string;
  hashValue: string;
  createdAt: string;
  ackId?: string;
  cino: string;               // CINO
  selected: boolean;          // selection flag
  posted: boolean;            // posted flag
  itemUUID: string;           // item UUID
  status?: 'idle' | 'submitting' | 'submitted' | 'error';
  checkStatusState?: 'idle' | 'checking' | 'checked' | 'error';
  userFriendlyPostResponse?: string;
  userFriendlyCheckResponse?: string;
  postResponse?: string;
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

  /** Generate a zip for an item UUID */
  generate(itemUUID: string): Observable<any> {
    return this.http.post<any>(`${this.baseUrl}/export/zip/${encodeURIComponent(itemUUID)}`, {});
  }

  /** Paginated generated-records list */
  getRecords(
    page: number = 0,
    size: number = 10,
    submitted?: 'submit' | 'notSubmitted',
    sortBy: string = 'createdAt',
    sortDir: 'asc' | 'desc' = 'desc'
  ): Observable<any> {
    let params = new HttpParams()
      .set('page', String(page))
      .set('size', String(size))
      .set('sortBy', sortBy)
      .set('sortDir', sortDir);

    if (submitted) {
      params = params.set('submitted', submitted);
    }

    return this.http.get<any>(`${this.baseUrl}/cnr/records`, { params });
  }

  /** Post (submit) a case by CNR */
  submitCase(cnr: string): Observable<any> {
    const params = new HttpParams().set('cnr', cnr);
    return this.http.post(`${this.baseUrl}/jtdr/submit`, null, { params });
  }

  /** Check JTDR status using ackId */
  checkStatus(ackId: string): Observable<any> {
    return this.http.get(`${this.baseUrl}/jtdr/status/${encodeURIComponent(ackId)}`);
  }

  /** Search (CINO or generic query) */
  getSearchResults(query: string): Observable<any> {
    const params = new HttpParams().set('query', query);
    return this.http.get<any>(`${this.baseUrl}/discover/search/objects`, { params });
  }

  /** Download report as CSV (already in your file) */
  getReportCsv(start: string, end: string): Observable<Blob> {
    const params = new HttpParams().set('start', start).set('end', end);
    return this.http.get(`${this.baseUrl}/jtdr/report`, {
      params,
      responseType: 'blob',
      headers: { Accept: 'text/csv' }
    });
  }

  /** NEW: Download report as PDF (component calls this) */
  getReportPdf(start: string, end: string): Observable<Blob> {
    const params = new HttpParams().set('start', start).set('end', end);
    return this.http.get(`${this.baseUrl}/jtdr/report`, {
      params,
      responseType: 'blob',
      headers: { Accept: 'application/pdf' }
    });
  }

  /** NEW: Delete a generated zip by its fileName shown in the list (component calls this) */
  deleteGenerated(fileName: string): Observable<void> {
    // If your backend expects query param instead, switch to the commented version below.
    return this.http.delete<void>(`${this.baseUrl}/export/zip/${encodeURIComponent(fileName)}`);
    // const params = new HttpParams().set('fileName', fileName);
    // return this.http.delete<void>(`${this.baseUrl}/export/zip`, { params });
  }
}






Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/
name of the file : app-router.ts







import type { InMemoryScrollingOptions, Route, RouterConfigOptions } from "@angular/router"

import { NOTIFICATIONS_MODULE_PATH } from "./admin/admin-routing-paths"
import {
  ACCESS_CONTROL_MODULE_PATH,
  ADMIN_MODULE_PATH,
  BITSTREAM_MODULE_PATH,
  ERROR_PAGE,
  FORBIDDEN_PATH,
  FORGOT_PASSWORD_PATH,
  HEALTH_PAGE_PATH,
  INFO_MODULE_PATH,
  INTERNAL_SERVER_ERROR,
  LEGACY_BITSTREAM_MODULE_PATH,
  PROFILE_MODULE_PATH,
  REGISTER_PATH,
  REQUEST_COPY_MODULE_PATH,
  WORKFLOW_ITEM_MODULE_PATH,
} from "./app-routing-paths"
import { COLLECTION_MODULE_PATH } from "./collection-page/collection-page-routing-paths"
import { COMMUNITY_MODULE_PATH } from "./community-page/community-page-routing-paths"
import { authBlockingGuard } from "./core/auth/auth-blocking.guard"
import { authenticatedGuard } from "./core/auth/authenticated.guard"
import { groupAdministratorGuard } from "./core/data/feature-authorization/feature-authorization-guard/group-administrator.guard"
import { siteAdministratorGuard } from "./core/data/feature-authorization/feature-authorization-guard/site-administrator.guard"
import { siteRegisterGuard } from "./core/data/feature-authorization/feature-authorization-guard/site-register.guard"
import { endUserAgreementCurrentUserGuard } from "./core/end-user-agreement/end-user-agreement-current-user.guard"
import { reloadGuard } from "./core/reload/reload.guard"
import { forgotPasswordCheckGuard } from "./core/rest-property/forgot-password-check-guard.guard"
import { ServerCheckGuard } from "./core/server-check/server-check.guard"
import { ThemedForbiddenComponent } from "./forbidden/themed-forbidden.component"
import { menuResolver } from "./menuResolver"
import { provideSuggestionNotificationsState } from "./notifications/provide-suggestion-notifications-state"
import { ThemedPageErrorComponent } from "./page-error/themed-page-error.component"
import { ThemedPageInternalServerErrorComponent } from "./page-internal-server-error/themed-page-internal-server-error.component"
import { ThemedPageNotFoundComponent } from "./pagenotfound/themed-pagenotfound.component"
import { PROCESS_MODULE_PATH } from "./process-page/process-page-routing.paths"
import { provideSubmissionState } from "./submission/provide-submission-state"
import { SUGGESTION_MODULE_PATH } from "./suggestions-page/suggestions-page-routing-paths"
import { SearchPageComponent } from "./case-search/search-page.component"
import { PetitionerRespondentSearchComponent } from "./petitioner-respondent-search/petitioner-respondent-search.component"
import { JudgementDateComponent } from "./judgment-date/judgement-date.component"
import { PetitionerRespondentSearchComponent3 } from "./petitioner-respondent-search copy 2/petitioner-respondent-search3.component"
import { FreeTextComponent } from "./judgment-date copy/free-text.component"
import { FreeTextComponent3 } from "./judgment-date copy 2/free-text3.component"
import { CaseDetailsComponent } from "./case-details/case-details.component"
import { JudgeNameComponent } from "./judgename/judgename.component"
import { ITEM_MODULE_PATH } from "./item-page/item-page-routing-paths"
import { FuzzySearchComponent } from "./fuzzy_search/free-text.component"
import { BooleanSearchComponent } from "./boolean-search/boolean-search.component"
import { ProximitySearchComponent } from "./proximity-search/proximity-search.component"
import { CnrManagerComponent } from "./cnr-manager/cnr-manager.component"

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
        path: "reload/:rnd",
        component: ThemedPageNotFoundComponent,
        pathMatch: "full",
        canActivate: [reloadGuard],
      },
      {
        path: "home",
        loadChildren: () => import("./home-page/home-page-routes").then((m) => m.ROUTES),
        data: { showBreadcrumbs: false, enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "community-list",
        loadChildren: () => import("./community-list-page/community-list-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "id",
        loadChildren: () => import("./lookup-by-id/lookup-by-id-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "handle",
        loadChildren: () => import("./lookup-by-id/lookup-by-id-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: REGISTER_PATH,
        loadChildren: () => import("./register-page/register-page-routes").then((m) => m.ROUTES),
        canActivate: [siteRegisterGuard],
      },
      {
        path: FORGOT_PASSWORD_PATH,
        loadChildren: () => import("./forgot-password/forgot-password-routes").then((m) => m.ROUTES),
        canActivate: [forgotPasswordCheckGuard],
      },
      {
        path: COMMUNITY_MODULE_PATH,
        loadChildren: () => import("./community-page/community-page-routes").then((m) => m.ROUTES),
        data: { enableRSS: true },
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: COLLECTION_MODULE_PATH,
        loadChildren: () => import("./collection-page/collection-page-routes").then((m) => m.ROUTES),
        data: { showBreadcrumbs: false, enableRSS: true },
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: ITEM_MODULE_PATH,
        loadChildren: () => import('./item-page/item-page-routes')
          .then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "entities/:entity-type",
        loadChildren: () => import("./item-page/item-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "viewer",
        loadChildren: () => import("./item-page/simple/field-components/viewer-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard],
      },
      {
        path: LEGACY_BITSTREAM_MODULE_PATH,
        loadChildren: () => import("./bitstream-page/bitstream-page-routes").then((m) => m.ROUTES),
        canActivate: [ endUserAgreementCurrentUserGuard],
      },
      {
        path: BITSTREAM_MODULE_PATH,
        loadChildren: () => import("./bitstream-page/bitstream-page-routes").then((m) => m.ROUTES),
        canActivate: [ endUserAgreementCurrentUserGuard],
      },
      {
        path: "mydspace",
        loadChildren: () => import("./my-dspace-page/my-dspace-page-routes").then((m) => m.ROUTES),
        data: { enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "adminpool",
        loadChildren: () => import("./admin-pool/admin-pool.module").then((m) => m.AdminPoolModule),
        data: { enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "search",
        loadChildren: () => import("./search-page/search-page-routes").then((m) => m.ROUTES),
        data: { enableRSS: true },
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },

      {
        path: "case-search",
        component: SearchPageComponent,
        canActivate: [authenticatedGuard],
      },
      {
        path: "jtdr",
        component: CnrManagerComponent,
        canActivate: [siteAdministratorGuard,endUserAgreementCurrentUserGuard],
      },
      {
        path: "judgement-date-search",
        component: JudgementDateComponent,
        canActivate: [authenticatedGuard],
      },

      {
        path: "case-search-2",
        component: PetitionerRespondentSearchComponent,
        canActivate: [authenticatedGuard],
      },

      {
        path: "free-text-search",
        component: PetitionerRespondentSearchComponent3,
        canActivate: [authenticatedGuard],
      },

      {
        path: "browse",
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
        children: [
          {
            path: "title",
            component: SearchPageComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_case_type",
            component: JudgeNameComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_case_number",
            component: PetitionerRespondentSearchComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_judge_name",
            component: JudgementDateComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_party_firstpetitioner",
            component: FreeTextComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_party_firstrespondent",
            component: FreeTextComponent3,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
           {
            path: "dc_advocate_firstpetitioner",
            component: BooleanSearchComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_advocate_firstrespondent",
            component: ProximitySearchComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "dc_case_disposaldate",
            component: FuzzySearchComponent,
            data: { enableRSS: true },
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
          {
            path: "",
            loadChildren: () => import("./browse-by/browse-by-page-routes").then((m) => m.ROUTES),
            canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
          },
        ],
      },
      {
        path: ADMIN_MODULE_PATH,
        loadChildren: () => import("./admin/admin-routes").then((m) => m.ROUTES),
        data: { enableRSS: true },
        canActivate: [siteAdministratorGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: NOTIFICATIONS_MODULE_PATH,
        loadChildren: () =>
          import("./quality-assurance-notifications-pages/notifications-pages-routes").then((m) => m.ROUTES),
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "login",
        loadChildren: () => import("./login-page/login-page-routes").then((m) => m.ROUTES),
      },
      {
        path: "logout",
        loadChildren: () => import("./logout-page/logout-page-routes").then((m) => m.ROUTES),
      },
      {
        path: "submit",
        loadChildren: () => import("./submit-page/submit-page-routes").then((m) => m.ROUTES),
        providers: [provideSubmissionState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "import-external",
        loadChildren: () => import("./import-external-page/import-external-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "workspaceitems",
        loadChildren: () => import("./workspaceitems-edit-page/workspaceitems-edit-page-routes").then((m) => m.ROUTES),
        providers: [provideSubmissionState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: WORKFLOW_ITEM_MODULE_PATH,
        providers: [provideSubmissionState()],
        loadChildren: () => import("./workflowitems-edit-page/workflowitems-edit-page-routes").then((m) => m.ROUTES),
        data: { enableRSS: true },
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: PROFILE_MODULE_PATH,
        loadChildren: () => import("./profile-page/profile-page-routes").then((m) => m.ROUTES),
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: PROCESS_MODULE_PATH,
        loadChildren: () => import("./process-page/process-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: SUGGESTION_MODULE_PATH,
        loadChildren: () => import("./suggestions-page/suggestions-page-routes").then((m) => m.ROUTES),
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: INFO_MODULE_PATH,
        loadChildren: () => import("./info/info-routes").then((m) => m.ROUTES),
      },
      {
        path: REQUEST_COPY_MODULE_PATH,
        loadChildren: () => import("./request-copy/request-copy-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: FORBIDDEN_PATH,
        component: ThemedForbiddenComponent,
      },
      {
        path: "statistics",
        loadChildren: () => import("./statistics-page/statistics-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: HEALTH_PAGE_PATH,
        loadChildren: () => import("./health-page/health-page-routes").then((m) => m.ROUTES),
      },
      {
        path: ACCESS_CONTROL_MODULE_PATH,
        loadChildren: () => import("./access-control/access-control-routes").then((m) => m.ROUTES),
        canActivate: [groupAdministratorGuard, endUserAgreementCurrentUserGuard],
      },
      {
        path: "subscriptions",
        loadChildren: () => import("./subscriptions-page/subscriptions-page-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard],
      },

      { path: "**", pathMatch: "full", component: ThemedPageNotFoundComponent },


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





Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/shared/auth-nav-menu/user-menu
Name of the file : user-menu.component.html



<ds-loading *ngIf="(loading$ | async)"></ds-loading>
<ul *ngIf="(loading$ | async) !== true" class="user-menu" role="menu"
  [ngClass]="inExpandableNavbar ? 'user-menu-mobile pb-2 mb-2 border-bottom' : 'user-menu-dropdown'"
  [attr.aria-label]="'nav.user-profile-menu-and-logout' |translate" id="user-menu-dropdown">
  <li class="ds-menu-item-wrapper username-email-wrapper" role="presentation">
    {{dsoNameService.getName(user$ | async)}}<br>
    <span class="text-muted">{{(user$ | async)?.email}}</span>
  </li>
  <li class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="[profileRoute]" routerLinkActive="active">{{'nav.profile' |
      translate}}</a>
  </li>
  <li class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="[mydspaceRoute]" routerLinkActive="active">{{'nav.mydspace' |
      translate}}</a>
  </li>
  <li class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="['/adminpool']" routerLinkActive="active">Admin pool</a>
  </li>
  <li *ngIf="isAdmin$ | async" class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="['/jtdr']" routerLinkActive="active">
      Jtdr
    </a>
  </li>

  <li class="ds-menu-item-wrapper" role="presentation" (click)="onMenuItemClick()">
    <a class="ds-menu-item" role="menuitem" [routerLink]="[subscriptionsRoute]"
      routerLinkActive="active">{{'nav.subscriptions' | translate}}</a>
  </li>
  <ng-container *ngIf="!inExpandableNavbar">
    <li class="dropdown-divider" aria-hidden="true"></li>
    <li class="ds-menu-item-wrapper" role="presentation">
      <ds-log-out data-test="log-out-component"></ds-log-out>
    </li>
  </ng-container>
</ul>



Location of the file : Dspace_Latest/dspace_frontend_latest-main/src/app/shared/auth-nav-menu/user-menu
Name of the file : user-menu.component.ts


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
import { AuthorizationDataService } from '../../../core/data/feature-authorization/authorization-data.service';
import { SiteDataService } from '../../../core/data/site-data.service';
import { switchMap } from 'rxjs/operators';
import { Site } from '../../../core/shared/site.model';
import { FeatureID } from '../../../core/data/feature-authorization/feature-id';


@Component({
  selector: 'ds-base-user-menu',
  templateUrl: './user-menu.component.html',
  styleUrls: ['./user-menu.component.scss'],
  standalone: true,
  imports: [NgIf, ThemedLoadingComponent, RouterLinkActive, NgClass, RouterLink, LogOutComponent, AsyncPipe, TranslateModule],
  
})
export class UserMenuComponent implements OnInit {


  @Input() inExpandableNavbar = false;


  @Output() changedRoute: EventEmitter<any> = new EventEmitter<any>();


  public loading$: Observable<boolean>;


  public user$: Observable<EPerson>;


  public mydspaceRoute = MYDSPACE_ROUTE;


  public profileRoute = getProfileModuleRoute();


  public isAdmin$: Observable<boolean>;

  public subscriptionsRoute = getSubscriptionsModuleRoute();

  constructor(
    protected store: Store<AppState>,
    protected authService: AuthService,
    public dsoNameService: DSONameService,
    private authorizationService: AuthorizationDataService,
    private siteDataService: SiteDataService


  ) {
  }


  ngOnInit(): void {

    this.loading$ = this.store.pipe(select(isAuthenticationLoading));

    this.user$ = this.authService.getAuthenticatedUserFromStore();
    
    this.isAdmin$ = this.siteDataService.find().pipe(
      switchMap((site: Site) =>
        this.authorizationService.isAuthorized(
          FeatureID.AdministratorOf,
          site._links.self.href
        )
      )
    );
    
    
        
    
    
    
    
    

  }

  /**
   * Emits an event when the menu item is clicked
   */
  onMenuItemClick() {
    this.changedRoute.emit();
  }
}
