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

      // {
      //   path: "items/:uuid",
      //   component: CaseDetailsComponent,
      //   canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard],
      // },
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
        path: 'user-audit',
        loadChildren: () => import("./admin-audit/admin-audit.module").then((m) => m.AdminAuditModule),
        data: { enableRSS: true },
        providers: [provideSuggestionNotificationsState()],
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard]
      },

      {
        path: 'admin-watermark',
        component: (AdminPannelComponent as any), // import the component at the top
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard]
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
        canActivate: [authenticatedGuard],
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

      // {
      //   path: "jtdr",
      //   loadChildren: ()=> import("./cnr-manager/cnr-manager.component").then((m)=>m.CnrManagerComponent)
      // },
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



import { Routes } from '@angular/router';
import { ViewerComponent } from './view-file-pdf/viewer.component';

export const ROUTES: Routes = [
  { path: 'i/:itemUuid/f/:bitstreamUuid', component: ViewerComponent },
];


import {
  Component,
  OnInit,
  ElementRef,
  ViewChild,
  AfterViewInit,
  ChangeDetectorRef,
  OnDestroy,
} from "@angular/core"
import { ActivatedRoute } from "@angular/router"
import { SignpostingDataService } from "src/app/core/data/signposting-data.service"
import { CommonModule } from "@angular/common"
import * as pdfjsLib from "pdfjs-dist"
import "pdfjs-dist/build/pdf.worker.entry"
import { ViewEncapsulation } from "@angular/core"
import { SignpostingDataService1 } from "src/app/core/serachpage/signposting-metadata-data.service"
import { Location } from "@angular/common"
import { HttpClient } from "@angular/common/http"
import { PdfService } from "src/app/core/serachpage/pdf-auth.service"
import {
  BitstreamPermissionsService,
  TimeAccessStatus,
} from "src/app/core/serachpage/bitstream-permissions.service"
import { interval, Subscription } from "rxjs"
import { FormsModule } from "@angular/forms"
import { BitstreamComment, BitstreamCommentService } from "src/app/core/serachpage/bitstream-comment.service"

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


  loadComments(bitstreamId: string): void {
    this.bitstreamCommentService.getComments(bitstreamId).subscribe({
      next: (res) => {
        this.comments = res
        this.cdr.detectChanges()
      },
      error: (err) => console.error("Failed to load comments", err),
    })
  }

  addComment(): void {
    const comment = this.newCommentText.trim()
    if (!comment || this.isAddingComment) return

    this.isAddingComment = true

    const newComment: BitstreamComment = {
      bitstreamId: this.currentBitstreamId,
      comment: comment,
    }

    this.bitstreamCommentService.addComment(newComment).subscribe({
      next: (res) => {
        this.comments.push(res)
        this.newCommentText = ""
        this.isAddingComment = false
        this.cdr.detectChanges()
      },
      error: (err) => {
        console.error("Failed to add comment", err)
        this.isAddingComment = false
        this.cdr.detectChanges()
      },
    })
  }

  // ✅ Custom confirmation modal methods
  confirmDelete(commentId: number): void {
    this.commentToDelete = commentId
    this.showDeleteConfirmation = true
  }

  cancelDelete(): void {
    this.showDeleteConfirmation = false
    this.commentToDelete = null
  }

  confirmDeleteComment(): void {
    if (this.commentToDelete === null) return

    this.deletingCommentId = this.commentToDelete
    this.showDeleteConfirmation = false

    this.bitstreamCommentService.deleteComment(this.commentToDelete).subscribe({
      next: () => {
        this.comments = this.comments.filter((c) => c.id !== this.commentToDelete)
        this.deletingCommentId = null
        this.commentToDelete = null
        this.cdr.detectChanges()
      },
      error: (err) => {
        console.error("Delete failed", err)
        this.deletingCommentId = null
        this.commentToDelete = null
        this.cdr.detectChanges()
      },
    })
  }
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



<div class="viewer-container">
  <!-- File Header -->
  <div class="file-header">
    <span class="filename">{{ fileUrl.split('/').pop() }}</span>
    <div class="header-buttons">
      <button class="icon-button" (click)="goBack()"><i class="back-icon">←</i> Back</button>
      <button class="icon-button" (click)="toggleFullScreen()" *ngIf="hasTimeAccess">
        Open in Full Screen
      </button>
    </div>
  </div>

  <!-- Main Content Layout -->
  <div class="content-container" style="display: flex; flex-direction: row; height: 100%;">

    <!-- LEFT: Metadata -->
    <div class="metadata-container" *ngIf="!isMetadataMinimized"
      style="width: 300px; background: white; overflow-y: auto; border-right: 1px solid #ddd;">
      <div class="metadata-header">
        <span>Metadata</span>
        <button class="icon-button panel-toggle" (click)="toggleMetadataPanel()" title="Minimize Metadata Panel"><i class="fas fa-chevron-left"></i></button>
      </div>
      <div class="metadata-table">
        <table>
          <tr>
            <th>Metadata</th>
            <th>Value</th>
          </tr>
          <tr *ngFor="let item of metadata">
            <td>{{ item.name }}</td>
            <td>{{ item.value }}</td>
          </tr>
        </table>
      </div>
    </div>
    <button *ngIf="isMetadataMinimized" class="icon-button panel-toggle minimized-left" (click)="toggleMetadataPanel()" title="Show Metadata"><i class="fas fa-chevron-right"></i></button>

    <!-- CENTER: File Viewer -->
    <div class="file-container" style="flex: 1; position: relative; overflow: hidden;">
      <!-- Loading -->
      <div *ngIf="isLoading || checkingPermissions" class="loading-overlay">
        <div class="spinner-container">
          <div class="spinner-border text-primary" role="status"><span class="visually-hidden">Loading...</span></div>
          <p class="mt-2">{{ checkingPermissions ? 'Checking permissions...' : 'Loading file...' }}</p>
        </div>
      </div>

      <!-- Time Access Denied -->
      <div *ngIf="!hasTimeAccess && !isLoading && !checkingPermissions" class="access-denied-container">
        <div class="access-denied-message">
          <h3>Access Restricted</h3>
          <p>{{ timeAccessStatus?.message || 'Access denied at this time.' }}</p>
          <button class="btn btn-primary mt-3" (click)="checkFilePermissions(currentBitstreamId)">Check Again</button>
        </div>
      </div>

      <!-- PDF Viewer -->
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

        <!-- Search Bar -->
        <div class="search-bar pdf-search-bar-image" *ngIf="isSearchVisible">
          <div class="search-input-container">
            <input id="pdf-search-input" [(ngModel)]="searchText" placeholder="Search in document..." (keyup.enter)="searchPdf()" aria-label="Search in document" />
            <button (click)="searchPdf()" title="Search" aria-label="Search"><i class="fas fa-search"></i></button>
            <button class="clear-button" (click)="clearSearch()" title="Clear search" aria-label="Clear search"><i class="fas fa-times"></i></button>
          </div>
          <div class="search-navigation" *ngIf="searchResults.length > 0">
            <span>{{ currentSearchIndex + 1 }} of {{ searchResults.length }}</span>
            <button class="icon-button" (click)="prevSearchResult()" [disabled]="currentSearchIndex <= 0" title="Previous result" aria-label="Previous result">
              <i class="fas fa-chevron-left"></i>
            </button>
            <button class="icon-button" (click)="nextSearchResult()" [disabled]="currentSearchIndex >= searchResults.length - 1" title="Next result" aria-label="Next result">
              <i class="fas fa-chevron-right"></i>
            </button>
          </div>
        </div>

        <!-- PDF Pages Container -->
        <div class="pdf-scroll-container pdf-scroll-image" #pdfContainer aria-label="PDF document area"></div>
      </div>

      <!-- Image Viewer -->
      <div class="image-container" *ngIf="isImageFile && hasTimeAccess">
        <!-- Image Toolbar -->
        <div class="toolbar toolbarimg">
          <div class="toolbar-left">
            <button class="icon-button" (click)="imageZoomOut()">−</button>
            <button class="icon-button" (click)="imageZoomIn()">+</button>
            <span>Zoom: {{ imageZoomLevel | number:'1.0-1' }}</span>
          </div>
          <div class="toolbar-right">
            <button class="icon-button" (click)="toggleImageFullScreen()">⛶</button>
            <button class="icon-button" (click)="openImageInNewTab()">↗</button>
            <button *ngIf="canDownloadFile" class="icon-button" (click)="downloadImage()">⬇</button>
          </div>
        </div>
        
        <!-- Image Display -->
        <div class="pdf-scroll-container">
          <img [src]="fileUrl" 
               class="responsive-image" 
               [style.transform]="'scale(' + imageZoomLevel + ')'"
               alt="Document Image" />
        </div>
      </div>

      <!-- Video Viewer -->
      <div class="video-container" *ngIf="isVideoFile && hasTimeAccess">
        <!-- Video Toolbar -->
        <div class="toolbar toolbar-video">
          <button class="icon-button" (click)="openVideoInNewTab()">↗</button>
          <button *ngIf="canDownloadFile" class="icon-button" (click)="downloadVideo()">⬇</button>
        </div>
        
        <!-- Video Player -->
        <div class="pdf-scroll-container">
          <video [src]="fileUrl" 
                 class="responsive-video" 
                 controls 
                 preload="metadata"
                 (error)="onVideoError($event)">
            Your browser does not support the video tag.
          </video>
          <div *ngIf="videoError" class="video-error-message">
            <p>⚠️ Unable to load video. Please check permissions or file format.</p>
            <p><strong>Debug info:</strong> {{ fileUrl }}</p>
          </div>
        </div>
      </div>

      <!-- Audio Viewer -->
      <div class="audio-container" *ngIf="isAudioFile && hasTimeAccess">
        <!-- Audio Toolbar -->
        <div class="toolbar toolbar-audio">
          <button class="icon-button" (click)="openAudioInNewTab()">↗</button>
          <button *ngIf="canDownloadFile" class="icon-button" (click)="downloadAudio()">⬇</button>
        </div>
        
        <!-- Audio Player -->
        <div class="pdf-scroll-container">
          <audio [src]="fileUrl" 
                 class="responsive-audio" 
                 controls 
                 preload="metadata">
            Your browser does not support the audio tag.
          </audio>
        </div>
      </div>

      <!-- Permission Notices -->
      <div *ngIf="!canDownloadFile && hasTimeAccess" class="permission-notice download-notice">
        ⚠️ Download not permitted for this file
      </div>
      
      <div *ngIf="!canPrintFile && hasTimeAccess" class="permission-notice print-notice">
        ⚠️ Printing not permitted for this file
      </div>
    </div>

    <!-- RIGHT: Comments Panel -->
    <div class="comment-panel" *ngIf="!isCommentMinimized">
      <h4>Comments</h4>

      <!-- No Comments Message -->
      <div *ngIf="comments.length === 0" class="no-comments">
        <p style="color: #666; font-style: italic;">No comments yet.</p>
      </div>

      <!-- Comments List -->
      <div *ngFor="let comment of comments" class="comment-box">
        <div class="comment-header">
          <span class="comment-author">{{ comment.commenterName || 'Anonymous' }}</span>
          <span class="comment-date">{{ comment.createdDate | date: 'medium' }}</span>
        </div>
        <div class="comment-body">
          <p>{{ comment.comment || comment.text }}</p>
        </div>
        
        <!-- Comment Actions (Only for Admins) -->
        <div class="comment-actions" *ngIf="isAdmin">
          <button class="delete-button" 
                  [disabled]="deletingCommentId === comment.id"
                  (click)="confirmDelete(comment.id!)">
            {{ deletingCommentId === comment.id ? 'Deleting...' : 'Delete' }}
          </button>
        </div>
      </div>

      <!-- Add Comment Form -->
      <div class="add-comment-box">
        <textarea [(ngModel)]="newCommentText" 
                  placeholder="Add comment..." 
                  [disabled]="!isAdmin || isAddingComment"
                  rows="3">
        </textarea>
        <button (click)="addComment()" 
                [disabled]="!isAdmin || isAddingComment || !newCommentText.trim()">
          {{ isAddingComment ? 'Adding...' : 'Add Comment' }}
        </button>
        
        <!-- Admin Notice -->
        <div *ngIf="!isAdmin" style="margin-top: 8px; font-size: 12px; color: #666; font-style: italic;">
          Only administrators can add comments.
        </div>
      </div>
    </div>
    <button *ngIf="isCommentMinimized" class="icon-button panel-toggle minimized-right" (click)="toggleCommentPanel()" title="Show Comments"><i class="fas fa-chevron-left"></i></button>

  </div>

  <!-- ✅ Custom Confirmation Modal -->
  <div class="modal-overlay" *ngIf="showDeleteConfirmation" (click)="cancelDelete()">
    <div class="confirmation-modal" (click)="$event.stopPropagation()">
      <div class="modal-header">
        <h3>Confirm Delete</h3>
      </div>
      <div class="modal-body">
        <p>Are you sure you want to delete this comment?</p>
        <p class="warning-text">This action cannot be undone.</p>
      </div>
      <div class="modal-footer">
        <button class="btn-cancel" (click)="cancelDelete()">Cancel</button>
        <button class="btn-delete" (click)="confirmDeleteComment()">Delete</button>
      </div>
    </div>
  </div>
</div>




import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from './api-urls';

export interface BitstreamComment {
  id?: number;
  bitstreamId: string;
  comment?: string; // request field
  text?: string;    // response field
}



@Injectable({ providedIn: 'root' })
export class BitstreamCommentService {
  private baseUrl = `${CURRENT_API_URL}/server/api/bitstream/comment`;

  constructor(private http: HttpClient) {}

  getComments(bitstreamId: string): Observable<BitstreamComment[]> {
    return this.http.get<BitstreamComment[]>(`${this.baseUrl}/bitstream/${bitstreamId}`, {
      withCredentials: true
    });
  }

  addComment(comment: BitstreamComment): Observable<BitstreamComment> {
    return this.http.post<BitstreamComment>(this.baseUrl, comment, {
      withCredentials: true
    });
  }

  updateComment(id: number, newText: string): Observable<BitstreamComment> {
    const headers = new HttpHeaders({ 'Content-Type': 'text/plain' });
    return this.http.put<BitstreamComment>(`${this.baseUrl}/${id}`, newText, {
      headers,
      withCredentials: true
    });
  }

  deleteComment(id: number): Observable<any> {
    return this.http.delete(`${this.baseUrl}/${id}`, {
      withCredentials: true
    });
  }
}




