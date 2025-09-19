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
import { AdminPannelComponent } from "./admin-pannel/admin-pannel.component"
// import { SearchByCaseComponent } from './searchN/search-by-case.component';

export const APP_ROUTES: Route[] = [
  { path: INTERNAL_SERVER_ERROR, component: ThemedPageInternalServerErrorComponent },
  { path: ERROR_PAGE, component: ThemedPageErrorComponent },
  {
    path: "",
    canActivate: [authBlockingGuard],
    canActivateChild: [ServerCheckGuard],
    resolve: [menuResolver],
    children: [

      {
        path: "viewer",
        loadChildren: () => import("./item-page/simple/field-components/viewer-routes").then((m) => m.ROUTES),
        canActivate: [authenticatedGuard],
      }
    ],
  },
]



import { Routes } from '@angular/router';
import { ViewerComponent } from './view-file-pdf/viewer.component';

export const ROUTES: Routes = [
  { path: 'i/:itemUuid/f/:bitstreamUuid', component: ViewerComponent },
];




import { PdfService } from "src/app/core/serachpage/pdf-auth.service"

@Component({
  selector: "app-viewer",
  templateUrl: "./viewer.component.html",
  styleUrls: ["./viewer.component.scss"],
  standalone: true,
  imports: [CommonModule, FormsModule],
  encapsulation: ViewEncapsulation.None,
})
export class ViewerComponent implements OnInit, AfterViewInit, OnDestroy {
  @ViewChild("pdfContainer", { static: false }) pdfContainer!: ElementRef<HTMLDivElement>
  isAdmin = false
  fileUrl = ""
  pdfDoc: any = null
  currentPage = 1
  totalPages = 0
  zoomLevel = 1.0
  fileType = ""
  isLoading = false
  currentBitstreamId = ""

  isPdfFile = false
  isImageFile = false
  isVideoFile = false
  isAudioFile = false

  isSearchVisible = false
  searchText = ""
  searchResults: any[] = []
  currentSearchIndex = -1

  searchTerm: string

  // Permission flags
  canDownloadFile = false
  canPrintFile = false
  checkingPermissions = false

  // Time-based access
  timeAccessStatus: TimeAccessStatus | null = null
  hasTimeAccess = false
  accessExpirationTimer: Subscription | null = null
  accessCheckInterval: Subscription | null = null

  // Fields to exclude from display
  excludedFields = ["dc.description.provenance", "dc.identifier.uri", "dc.date.accessioned"]

  comments: BitstreamComment[] = []
  newCommentText = ""
  isAddingComment = false

  // ✅ Custom confirmation modal properties
  showDeleteConfirmation = false
  commentToDelete: number | null = null
  deletingCommentId: number | null = null

  // Initial metadata array (will be replaced with API data)
  metadata: { name: string; value: string }[] = []

  // Ordered list of metadata keys to control display order
  orderedMetadataKeys = [
    "dc.casetype", // 1. Case Type
    "dc.title", // 2. Case Number
    "dc.caseyear", // 3. Case Year
    "dc.date.disposal", // 4. Disposal Date
    "dc.contributor.author", // 5. Judge Name
    "dc.pname", // 6. Petitioner Name
    "dc.rname", // 7. Respondent Name
    "dc.paname", // 8. Petitioner's Advocate Name
    "dc.raname", // 9. Respondent's Advocate Name
    "dc.district", // 10. District
    "dc.date.scan", // 11. Scan Date
    "dc.verified-by", // 12. Verified By
    "dc.date.verification", // 13. Date Verification
    "dc.barcode", // 14. Barcode Number
    "dc.batch-number", // 15. Batch Number
    "dc.size", // 16. File Size
    "dc.char-count", // 17. Character Count
    "dc.pages", // 18. No of Pages of the Main File
  ]

  customMetadataLabels: { [key: string]: string } = {
    "dc.caseyear": "Case Year",
    "dc.casetype": "Case Type",
    "dc.title": "Case Number",
    "dc.case.district": "District",
    "dc.district": "District",
    "dc.pname": "Petitioner Name",
    "dc.rname": "Respondent Name",
    "dc.paname": "Petitioner's Advocate Name",
    "dc.raname": "Respondent's Advocate Name",
    "dc.contributor.author": "Judge Name",
    "dc.date.accessioned": "Access Date",
    "dc.date.issued": "Issued Year",
    "dc.date.disposal": "Disposal Date",
    "dc.identifier.uri": "Handle URL",
    "dc.title.alternative": "Alternative Title",
    "dc.type": "Document Type",
    "dc.barcode": "Barcode Number",
    "dc.batch-number": "Batch Number",
    "dc.char-count": "Character Count",
    "dc.date.scan": "Scan Date",
    "dc.date.verification": "Date Verification",
    "dc.pages": "No of Pages of the Main File",
    "dc.size": "File Size",
    "dc.verified-by": "Verified By",
  }

  isMetadataMinimized = false;
  isCommentMinimized = false;

  videoError = false;

  constructor(

    private pdfService: PdfService,

  ) { }

  private fetchRestrictedPdf(bitstreamUuid: string): void {
    this.isLoading = true

    this.pdfService.fetchRestrictedPdf(bitstreamUuid).subscribe({
      next: (blob) => {
        const blobUrl = this.pdfService.createBlobUrl(blob)
        this.fileUrl = blobUrl
        this.loadPdf() // this uses fileUrl internally
        this.isLoading = false
        this.cdr.detectChanges()
      },
      error: (err) => {
        console.error("❌ Error fetching restricted PDF:", err)
        this.isLoading = false
        this.cdr.detectChanges()
      },
    })
  }

    loadPdf(): void {
    pdfjsLib
      .getDocument({ url: this.fileUrl })
      .promise.then((pdf) => {
        this.pdfDoc = pdf
        this.totalPages = pdf.numPages
        this.currentPage = 1
        this.renderAllPages()
        this.cdr.detectChanges()
      })
      .catch((err) => console.error("PDF loading error:", err))
  }

}

<div class="viewer-container">

      <div class="pdf-container pdf-image-style" *ngIf="isPdfFile && hasTimeAccess">
        <!-- Time Access Info Banner (always visible above toolbar) -->
        <div *ngIf="hasTimeAccess && timeAccessStatus?.validUntil" class="access-info-banner">
          <i class="fas fa-clock"></i>
          ⏰ Access expires: {{ timeAccessStatus.validUntil | date:'medium' }}
        </div>
        <!-- PDF Toolbar/Header (matches image) -->
        <div class="toolbar pdf-toolbar-image pdf-toolbar-header" role="toolbar" aria-label="PDF toolbar">
          <button class="icon-button panel-toggle" (click)="toggleMetadataPanel()" [ngClass]="{'minimized': isMetadataMinimized}" title="{{ isMetadataMinimized ? 'Show Metadata' : 'Minimize Metadata Panel' }}">
            <i class="fas" [ngClass]="isMetadataMinimized ? 'fa-chevron-right' : 'fa-chevron-left'"></i>
          </button>
          <div class="pdf-toolbar-center-group">
            <button class="icon-button" (click)="maximizePdfView()" *ngIf="!isMetadataMinimized || !isCommentMinimized" title="Maximize PDF View"><i class="fas fa-expand"></i></button>
            <button class="icon-button" (click)="restorePanels()" *ngIf="isMetadataMinimized && isCommentMinimized" title="Restore Panels"><i class="fas fa-compress"></i></button>
            <!-- PDF toolbar controls (page nav, zoom, etc) -->
            <div class="toolbar-left toolbar-group">
              <div class="page-navigation" aria-label="Page navigation">
                <button class="arrow-button up-arrow" (click)="prevPage()" [disabled]="currentPage <= 1" title="Previous Page" aria-label="Previous Page">
                  <i class="fas fa-chevron-up"></i>
                </button>
                <div class="page-number-box" aria-label="Current Page">{{ currentPage }}</div>
                <button class="arrow-button down-arrow" (click)="nextPage()" [disabled]="currentPage >= totalPages" title="Next Page" aria-label="Next Page">
                  <i class="fas fa-chevron-down"></i>
                </button>
                <span class="page-separator">of {{ totalPages }}</span>
              </div>
              <button class="icon-button" (click)="toggleSearch()" title="Search in PDF" aria-label="Search in PDF">
                <i class="fas fa-search"></i>
              </button>
            </div>
            <div class="toolbar-center toolbar-group">
              <button class="icon-button" (click)="zoomOut()" title="Zoom Out" aria-label="Zoom Out">
                <i class="fas fa-search-minus"></i>
              </button>
              <span class="zoom-level">{{ zoomLevel | number:'1.0-1' }}x</span>
              <button class="icon-button" (click)="zoomIn()" title="Zoom In" aria-label="Zoom In">
                <i class="fas fa-search-plus"></i>
              </button>
            </div>
            <div class="toolbar-right toolbar-group">
              <button *ngIf="canPrintFile" class="icon-button" (click)="printFile()" title="Print PDF" aria-label="Print PDF">
                <i class="fas fa-print"></i>
              </button>
              <button *ngIf="canDownloadFile" class="icon-button" (click)="downloadFile()" title="Download PDF" aria-label="Download PDF">
                <i class="fas fa-download"></i>
              </button>
            </div>
          </div>
          <button class="icon-button panel-toggle" (click)="toggleCommentPanel()" [ngClass]="{'minimized': isCommentMinimized}" title="{{ isCommentMinimized ? 'Show Comments' : 'Minimize Comment Panel' }}">
            <i class="fas" [ngClass]="isCommentMinimized ? 'fa-chevron-left' : 'fa-chevron-right'"></i>
          </button>
        </div>

  </div>

</div>




.viewer-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
}



import { NgModule } from "@angular/core"
import { CommonModule } from "@angular/common"
import { SafePipeModule } from "./safe-pipe.module"

@NgModule({
  declarations: [],
  imports: [CommonModule, SafePipeModule],
  exports: [],
})
export class ViewerModule {}



import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';
import { CURRENT_API_URL } from './api-urls';

@Injectable({
  providedIn: 'root'
})
export class PdfService {
  constructor(private http: HttpClient) {}

  /**
   * Fetches a restricted PDF from the server API
   * @param bitstreamUuid The UUID of the bitstream to fetch
   * @returns An Observable of the PDF blob
   */
  fetchRestrictedPdf(bitstreamUuid: string): Observable<Blob> {
    const url = `${CURRENT_API_URL}/server/api/custom/bitstreams/${bitstreamUuid}/filtered-content`;
    
    return this.http.get(url, { 
      responseType: 'blob', 
      withCredentials: true 
    }).pipe(
      retry(2), // Retry failed requests up to 2 times
      catchError(error => {
        console.error('Error fetching PDF:', error);
        return throwError(() => new Error('Failed to fetch PDF. Please try again later.'));
      })
    );
  }

  /**
   * Revokes a blob URL to free up memory
   * @param url The blob URL to revoke
   */
  revokeBlobUrl(url: string): void {
    if (url && url.startsWith('blob:')) {
      URL.revokeObjectURL(url);
    }
  }
}