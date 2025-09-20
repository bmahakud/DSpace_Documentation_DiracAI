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
    private route: ActivatedRoute,
    private signpostingService: SignpostingDataService,
    private metadataApiService: SignpostingDataService1,
    private location: Location,
    private cdr: ChangeDetectorRef,
    private http: HttpClient,
    private pdfService: PdfService,
    private bitstreamPermissionsService: BitstreamPermissionsService,
    private bitstreamCommentService: BitstreamCommentService,
  ) { }

  ngOnInit(): void {
    this.route.params.subscribe((params) => {
      const itemUuid = params["itemUuid"]
      const bitstreamUuid = params["bitstreamUuid"]

      if (itemUuid && bitstreamUuid) {
        this.currentBitstreamId = bitstreamUuid
        this.fetchMetadataFromApi(itemUuid)
        this.checkFilePermissions(bitstreamUuid)
      } else {
        console.error("❌ Missing itemUuid or bitstreamUuid in route")
      }
    })
  }

  checkFilePermissions(bitstreamId: string): void {
    this.checkingPermissions = true

    // Step 1: Fetch all permission info (includes isAdmin flag)
    this.bitstreamPermissionsService.getBitstreamPermissions(bitstreamId).subscribe({
      next: (permData) => {
        this.isAdmin = permData?.isAdmin === true

        if (this.isAdmin) {
          // ✅ Full access for admins
          this.canDownloadFile = true
          this.canPrintFile = true
          this.hasTimeAccess = true
          document.body.classList.add("can-print")

          const itemUuid = this.route.snapshot.params["itemUuid"]
          this.fetchFileData(itemUuid, bitstreamId)

          this.checkingPermissions = false
          this.cdr.detectChanges()
          return // ✅ Skip further permission/time checks
        }

        // ✅ Step 2: Check time-based access for non-admins
        this.bitstreamPermissionsService.checkTimeAccess(bitstreamId).subscribe({
          next: (accessStatus) => {
            this.timeAccessStatus = accessStatus
            this.hasTimeAccess = accessStatus.hasAccess

            console.log(`Time-based access for ${bitstreamId}:`, accessStatus)

            if (this.hasTimeAccess) {
              const itemUuid = this.route.snapshot.params["itemUuid"]
              this.fetchFileData(itemUuid, bitstreamId)

              this.setupAccessExpirationTimer()
              this.setupPeriodicAccessCheck(bitstreamId)
            }

            // ✅ Step 3: Download permission
            this.bitstreamPermissionsService.canDownload(bitstreamId).subscribe({
              next: (canDownload) => {
                this.canDownloadFile = canDownload
              },
              error: (err) => {
                console.error("Error checking download permission:", err)
                this.canDownloadFile = false
              },
            })

            // ✅ Step 4: Print permission
            this.bitstreamPermissionsService.canPrint(bitstreamId).subscribe({
              next: (canPrint) => {
                this.canPrintFile = canPrint
                if (canPrint) {
                  document.body.classList.add("can-print")
                } else {
                  document.body.classList.remove("can-print")
                }
              },
              error: (err) => {
                console.error("Error checking print permission:", err)
                this.canPrintFile = false
                document.body.classList.remove("can-print")
              },
              complete: () => {
                this.checkingPermissions = false
                this.cdr.detectChanges()
              },
            })
          },
          error: (err) => {
            console.error("Error checking time-based access:", err)
            this.hasTimeAccess = false
            this.timeAccessStatus = {
              hasAccess: false,
              message: "Error checking access permissions.",
              validUntil: null,
              validFrom: null,
            }
            this.checkingPermissions = false
            this.cdr.detectChanges()
          },
        })
      },
      error: (err) => {
        console.error("Error fetching bitstream permissions:", err)
        this.checkingPermissions = false
        this.cdr.detectChanges()
      },
    })
  }

  setupAccessExpirationTimer(): void {
    // Clear any existing timer
    if (this.accessExpirationTimer) {
      this.accessExpirationTimer.unsubscribe()
      this.accessExpirationTimer = null
    }

    // If we have an expiration time, set up a timer to check when access expires
    if (this.timeAccessStatus?.validUntil) {
      const now = new Date()
      const expirationTime = this.timeAccessStatus.validUntil
      const timeUntilExpiration = expirationTime.getTime() - now.getTime()

      if (timeUntilExpiration > 0) {
        console.log(`Setting up access expiration timer for ${timeUntilExpiration}ms`)

        // Set up timer to check access when it expires
        this.accessExpirationTimer = interval(timeUntilExpiration).subscribe(() => {
          console.log("Access expiration timer triggered")
          this.checkFilePermissions(this.currentBitstreamId)
        })
      }
    }
  }

  setupPeriodicAccessCheck(bitstreamId: string): void {
    // Clear any existing interval
    if (this.accessCheckInterval) {
      this.accessCheckInterval.unsubscribe()
      this.accessCheckInterval = null
    }

    // Set up periodic check (every minute) to ensure access is still valid
    this.accessCheckInterval = interval(60000).subscribe(() => {
      console.log("Periodic access check triggered")
      this.bitstreamPermissionsService.checkTimeAccess(bitstreamId).subscribe({
        next: (accessStatus) => {
          // If access status has changed, update UI
          if (this.hasTimeAccess !== accessStatus.hasAccess) {
            this.timeAccessStatus = accessStatus
            this.hasTimeAccess = accessStatus.hasAccess

            if (!this.hasTimeAccess) {
              // Access has been revoked, clean up resources
              if (this.fileUrl && this.fileUrl.startsWith("blob:")) {
                this.pdfService.revokeBlobUrl(this.fileUrl)
                this.fileUrl = ""
              }
              this.pdfDoc = null
            }

            this.cdr.detectChanges()
          }
        },
      })
    })
  }

  fetchMetadataFromApi(uuid: string): void {
    this.metadataApiService.getItemByUuid(uuid).subscribe({
      next: (res) => {
        const metadataMap: { [key: string]: string } = {}
        const apiMetadata = res.metadata

        // First, extract all metadata into a map, excluding the specified fields
        for (const key in apiMetadata) {
          if (apiMetadata.hasOwnProperty(key) && !this.excludedFields.includes(key)) {
            const values = apiMetadata[key]
            const combinedValue = values.map((v: any) => v.value).join(", ")
            metadataMap[key] = combinedValue
          }
        }

        // Then create the metadata array in the specified order
        const orderedMetadata: { name: string; value: string }[] = []

        // First add the ordered keys
        this.orderedMetadataKeys.forEach((key) => {
          if (metadataMap[key]) {
            orderedMetadata.push({
              name: this.customMetadataLabels[key] || key,
              value: metadataMap[key],
            })
          }
        })

        // Then add any remaining metadata not in the ordered list and not excluded
        for (const key in metadataMap) {
          if (!this.orderedMetadataKeys.includes(key)) {
            orderedMetadata.push({
              name: this.customMetadataLabels[key] || key,
              value: metadataMap[key],
            })
          }
        }

        this.metadata = orderedMetadata

        // Log the processed metadata
        console.log("Processed metadata:", this.metadata)

        // Log each metadata field individually for better readability
        console.log("Metadata fields:")
        this.metadata.forEach((item) => {
          console.log(`${item.name}: ${item.value}`)
        })

        this.cdr.detectChanges()
      },
      error: (err) => {
        console.error("Error fetching metadata from API", err)
      },
    })
  }

  goBack(): void {
    this.location.back()
  }

  fetchFileData(itemUuid: string, bitstreamUuid: string): void {
    this.videoError = false; // Reset error state on new file
    this.loadComments(bitstreamUuid)

    this.signpostingService.getLinks(itemUuid).subscribe((links) => {
      if (links && links.length > 0) {
        const fileItem = links.find((item) => item.rel === "item" && item.href.includes(bitstreamUuid))

        if (fileItem) {
          this.fileType = fileItem.type
          const lowerUrl = fileItem.href.toLowerCase()

          // Improved file type detection
          this.isPdfFile = this.fileType === "application/pdf" || lowerUrl.endsWith(".pdf")
          this.isImageFile = this.fileType.startsWith("image/") || [".jpg", ".jpeg", ".png", ".gif", ".bmp", ".webp", ".tiff", ".svg"].some(ext => lowerUrl.endsWith(ext))
          this.isVideoFile = this.fileType.startsWith("video/") || [".mp4", ".webm", ".ogg", ".mov", ".avi", ".mkv"].some(ext => lowerUrl.endsWith(ext))
          this.isAudioFile = this.fileType.startsWith("audio/") || [".mp3", ".wav", ".m4a", ".ogg", ".aac", ".flac"].some(ext => lowerUrl.endsWith(ext))

          // ✅ Load PDF via secure filtered-content endpoint
          if (this.isPdfFile) {
            this.fetchRestrictedPdf(bitstreamUuid) // Replaces direct fileUrl usage
          } else {
            this.fileUrl = fileItem.href
            this.cdr.detectChanges() // Moved here so it doesn't run before PDF blob is set
          }
        } else {
          console.error("❌ File with given bitstream UUID not found in signposting links")
        }
      } else {
        console.error("❌ No signposting links found for this item")
      }
    })
  }

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

  renderAllPages(): void {
    const container = this.pdfContainer.nativeElement
    container.innerHTML = ""

    for (let pageNum = 1; pageNum <= this.totalPages; pageNum++) {
      this.pdfDoc.getPage(pageNum).then((page: any) => {
        const viewport = page.getViewport({ scale: this.zoomLevel })

        // Page container
        const pageContainer = document.createElement("div")
        pageContainer.classList.add("page-container")
        pageContainer.style.position = "relative"
        pageContainer.style.margin = "16px auto"
        pageContainer.dataset.pageNumber = pageNum.toString()
        pageContainer.style.width = `${viewport.width}px`
        pageContainer.style.height = `${viewport.height}px`

        // Canvas
        const canvas = document.createElement("canvas")
        canvas.width = viewport.width
        canvas.height = viewport.height
        const ctx = canvas.getContext("2d")!
        page.render({ canvasContext: ctx, viewport })

        pageContainer.appendChild(canvas)
        container.appendChild(pageContainer)

        // Text Layer
        page.getTextContent().then((textContent: any) => {
          const textLayerDiv = document.createElement("div")
          textLayerDiv.className = "textLayer"
          textLayerDiv.style.position = "absolute"
          textLayerDiv.style.top = "0"
          textLayerDiv.style.left = "0"
          textLayerDiv.style.height = `${viewport.height}px`
          textLayerDiv.style.width = `${viewport.width}px`
          textLayerDiv.style.pointerEvents = "none"

          pageContainer.appendChild(textLayerDiv)
            ; (pdfjsLib as any)
              .renderTextLayer({
                textContent,
                container: textLayerDiv,
                viewport,
                textDivs: [],
              })
              .promise.then(() => {
                const termToHighlight = this.searchText?.trim() || this.searchTerm?.trim()
                if (termToHighlight) {
                  this.highlightMatches(textLayerDiv, termToHighlight)
                }
              })
        })
      })
    }
  }

  highlightMatches(textLayerDiv: HTMLElement, searchTerm: string): void {
    if (!searchTerm || searchTerm.trim() === "") return

    const normalizedSearch = searchTerm.trim().replace(/[-/\\^$*+?.()|[\]{}]/g, "\\$&")
    const regex = new RegExp(normalizedSearch, "gi")

    // Remove old highlights
    const oldHighlights = textLayerDiv.querySelectorAll("span mark.highlight-search")
    oldHighlights.forEach((mark) => {
      const parent = mark.parentNode as HTMLElement
      if (parent) {
        parent.replaceChild(document.createTextNode(mark.textContent || ""), mark)
        parent.normalize()
      }
    })

    const spans = textLayerDiv.querySelectorAll("span")
    spans.forEach((span) => {
      const text = span.textContent || ""
      if (!regex.test(text)) return

      // Reset regex state for reuse
      regex.lastIndex = 0

      const frag = document.createDocumentFragment()
      let lastIndex = 0
      let match: RegExpExecArray | null

      while ((match = regex.exec(text)) !== null) {
        const start = match.index
        const end = start + match[0].length

        // Add unhighlighted text
        if (lastIndex < start) {
          frag.appendChild(document.createTextNode(text.slice(lastIndex, start)))
        }

        // Add highlighted match
        const highlight = document.createElement("mark")
        highlight.className = "highlight-search"
        highlight.textContent = match[0]
        frag.appendChild(highlight)

        lastIndex = end
      }

      // Add remaining text
      if (lastIndex < text.length) {
        frag.appendChild(document.createTextNode(text.slice(lastIndex)))
      }

      span.innerHTML = ""
      span.appendChild(frag)
    })
  }

  nextPage(): void {
    if (this.currentPage < this.totalPages) {
      this.currentPage++
      this.scrollToPage(this.currentPage)
    }
  }

  prevPage(): void {
    if (this.currentPage > 1) {
      this.currentPage--
      this.scrollToPage(this.currentPage)
    }
  }

  scrollToPage(pageNumber: number): void {
    const container = this.pdfContainer.nativeElement
    const pages = container.querySelectorAll(".page-container")

    if (pages[pageNumber - 1]) {
      const offsetTop = (pages[pageNumber - 1] as HTMLElement).offsetTop
      container.scrollTo({ top: offsetTop - 20, behavior: "smooth" })
    }
  }

  updateCurrentPageOnScroll(): void {
    const container = this.pdfContainer.nativeElement
    const pages = Array.from(container.querySelectorAll(".page-container"))

    for (let i = 0; i < pages.length; i++) {
      const page = pages[i] as HTMLElement
      const offsetTop = page.offsetTop
      const pageHeight = page.offsetHeight

      if (container.scrollTop < offsetTop + pageHeight / 2) {
        this.currentPage = i + 1
        this.cdr.detectChanges()
        break
      }
    }
  }

  ngAfterViewInit(): void {
    if (this.pdfContainer && this.pdfContainer.nativeElement) {
      this.pdfContainer.nativeElement.addEventListener("scroll", () => {
        this.updateCurrentPageOnScroll()
      })
    }
  }

  zoomIn(): void {
    this.zoomLevel += 0.2
    this.renderAllPages()
  }

  zoomOut(): void {
    if (this.zoomLevel > 0.5) {
      this.zoomLevel -= 0.2
      this.renderAllPages()
    }
  }

  toggleFullScreen(): void {
    const elem = document.querySelector(".file-container")
    if (elem && !document.fullscreenElement) {
      ; (elem as HTMLElement).requestFullscreen()
    } else {
      document.exitFullscreen()
    }
  }

  openInNewTab(): void {
    window.open(this.fileUrl, "_blank")
  }

  printFile(): void {
    if (!this.canPrintFile) {
      console.warn("Print permission denied")
      return
    }
    // Open the actual PDF in a new window and trigger print
    const pdfWindow = window.open(this.fileUrl, '_blank');
    if (pdfWindow) {
      // Wait for the PDF to load, then print
      const printListener = () => {
        pdfWindow.focus();
        pdfWindow.print();
        pdfWindow.removeEventListener('load', printListener);
      };
      pdfWindow.addEventListener('load', printListener);
    } else {
      alert('Popup blocked! Please allow popups for this site to print the PDF.');
    }
  }

  downloadFile(): void {
    if (!this.canDownloadFile) {
      console.warn("Download permission denied");
      return;
    }

    const filename = this.generateCustomFilename() || "encrypted.pdf";

    this.pdfService.encryptBitstream(this.currentBitstreamId).subscribe({
      next: (blob) => {
        const blobUrl = this.pdfService.createBlobUrl(blob);
        const a = document.createElement('a');
        a.href = blobUrl;
        a.download = filename;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        this.pdfService.revokeBlobUrl(blobUrl);

        console.log("🔐 Encrypted file downloaded:", filename);
      },
      error: (err) => {
        console.error("❌ Error downloading encrypted file:", err);
      }
    });
  }

  private generateCustomFilename(): string {
    const caseType = this.findMetadataByPartialName("type")
    const caseNumber = this.findMetadataByPartialName("number")
    const caseYear = this.findMetadataByPartialName("year")

    const parts = [caseType, caseNumber, caseYear]
      .map((p) => p?.replace(/[^a-zA-Z0-9]/g, ""))
      .filter(Boolean)

    return parts.length > 0 ? parts.join("_") + ".pdf" : "encrypted.pdf"
  }




  // Helper method to find metadata by partial name match (case insensitive)
  findMetadataByPartialName(partialName: string): string {
    const lowerPartialName = partialName.toLowerCase()

    // Find any metadata entry where the name contains the partial name
    const entry = this.metadata.find((item) => item.name.toLowerCase().includes(lowerPartialName))

    return entry?.value?.trim() || ""
  }

  imageZoomLevel = 1.0

  imageZoomIn(): void {
    this.imageZoomLevel += 0.2
  }

  imageZoomOut(): void {
    if (this.imageZoomLevel > 0.4) {
      this.imageZoomLevel -= 0.2
    }
  }

  downloadImage(): void {
    if (!this.canDownloadFile) {
      console.warn("Download permission denied")
      return
    }

    const img = new Image()
    img.crossOrigin = "anonymous"
    img.src = this.fileUrl

    img.onload = () => {
      const canvas = document.createElement("canvas")
      canvas.width = img.naturalWidth
      canvas.height = img.naturalHeight
      const ctx = canvas.getContext("2d")
      ctx?.drawImage(img, 0, 0)

      canvas.toBlob((blob) => {
        if (blob) {
          const url = URL.createObjectURL(blob)
          const a = document.createElement("a")
          a.href = url
          a.download = this.fileUrl.split("/").pop() || "image.jpg"
          document.body.appendChild(a)
          a.click()
          document.body.removeChild(a)
          URL.revokeObjectURL(url)
        }
      }, "image/jpeg")
    }

    img.onerror = () => {
      console.error("Image failed to load. Cannot download.")
    }
  }

  openImageInNewTab(): void {
    window.open(this.fileUrl, "_blank")
  }

  toggleImageFullScreen(): void {
    const elem = document.querySelector(".image-container")
    if (elem && !document.fullscreenElement) {
      ; (elem as HTMLElement).requestFullscreen()
    } else {
      document.exitFullscreen()
    }
  }

  toggleSearch(): void {
    this.isSearchVisible = !this.isSearchVisible
    this.cdr.detectChanges()
    if (this.isSearchVisible) {
      setTimeout(() => {
        const searchInput = document.getElementById("pdf-search-input")
        if (searchInput) {
          searchInput.focus()
        }
      }, 100)
    }
  }

  onSearchInput(event: Event): void {
    this.searchText = (event.target as HTMLInputElement).value
  }

  clearSearch(): void {
    this.searchText = ""
    this.searchResults = []
    this.currentSearchIndex = -1

    const container = this.pdfContainer?.nativeElement
    if (!container) return

    const textLayers = container.querySelectorAll(".textLayer")

    textLayers.forEach((layer) => {
      const oldMarks = layer.querySelectorAll("mark.highlight-search")
      oldMarks.forEach((mark) => {
        const parent = mark.parentNode
        if (parent) {
          parent.replaceChild(document.createTextNode(mark.textContent || ""), mark)
          parent.normalize()
        }
      })
    })

    this.cdr.detectChanges()
  }

  searchPdf(): void {
    if (!this.searchText.trim() || !this.pdfDoc) return

    this.searchResults = []
    this.currentSearchIndex = -1

    const searchPromises = []

    for (let i = 1; i <= this.totalPages; i++) {
      searchPromises.push(
        this.pdfDoc.getPage(i).then((page: any) => {
          return page.getTextContent().then((textContent: any) => {
            const text = textContent.items.map((item: any) => item.str).join(" ")
            const regex = new RegExp(this.searchText, "gi")
            let match

            while ((match = regex.exec(text)) !== null) {
              this.searchResults.push({
                pageNum: i,
                position: match.index,
                text: match[0],
              })
            }
          })
        }),
      )
    }

    Promise.all(searchPromises).then(() => {
      if (this.searchResults.length > 0) {
        this.currentSearchIndex = 0

        // 👇 First render all pages, THEN scroll to first match
        this.renderAllPages()

        // 👇 Wait for rendering to complete before scrolling
        setTimeout(() => {
          this.navigateToSearchResult(0)
        }, 300) // Delay ensures DOM is ready
      }

      this.cdr.detectChanges()
    })
  }

  navigateToSearchResult(index: number): void {
    if (index >= 0 && index < this.searchResults.length) {
      const result = this.searchResults[index]
      this.currentSearchIndex = index

      const container = this.pdfContainer.nativeElement
      const pageContainers = container.querySelectorAll(".page-container")
      const targetPage = pageContainers[result.pageNum - 1] as HTMLElement

      if (targetPage) {
        container.scrollTo({
          top: targetPage.offsetTop - 50,
          behavior: "smooth",
        })

        const oldActives = container.querySelectorAll(".highlight-search.active")
        oldActives.forEach((el) => el.classList.remove("active"))

        const textLayers = container.querySelectorAll(".textLayer")
        textLayers.forEach((layer) => {
          this.highlightMatches(layer as HTMLElement, this.searchText)
        })

        const currentTextLayer = targetPage.querySelector(".textLayer")
        if (currentTextLayer) {
          const matches = currentTextLayer.querySelectorAll(".highlight-search")
          if (matches.length > 0) {
            ; (matches[0] as HTMLElement).classList.add("active")
          }
        }
      }
    }
  }

  nextSearchResult(): void {
    if (this.currentSearchIndex < this.searchResults.length - 1) {
      this.navigateToSearchResult(this.currentSearchIndex + 1)
    }
  }

  prevSearchResult(): void {
    if (this.currentSearchIndex > 0) {
      this.navigateToSearchResult(this.currentSearchIndex - 1)
    }
  }

  downloadVideo(): void {
    if (!this.canDownloadFile) {
      console.warn("Download permission denied")
      return
    }

    fetch(this.fileUrl)
      .then((res) => {
        if (!res.ok) throw new Error("Network response was not ok")
        return res.blob()
      })
      .then((blob) => {
        const url = URL.createObjectURL(blob)
        const a = document.createElement("a")
        a.href = url
        a.download = this.fileUrl.split("/").pop() || "video.mp4"
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      })
      .catch((err) => {
        console.error("Download error:", err)
      })
  }

  openVideoInNewTab(): void {
    window.open(this.fileUrl, "_blank")
  }

  openAudioInNewTab(): void {
    window.open(this.fileUrl, "_blank")
  }

  downloadAudio(): void {
    if (!this.canDownloadFile) {
      console.warn("Download permission denied")
      return
    }

    fetch(this.fileUrl)
      .then((res) => {
        if (!res.ok) throw new Error("Network response was not ok")
        return res.blob()
      })
      .then((blob) => {
        const url = URL.createObjectURL(blob)
        const a = document.createElement("a")
        a.href = url
        a.download = this.fileUrl.split("/").pop() || "audio.mp3"
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      })
      .catch((err) => {
        console.error("Download error:", err)
      })
  }

  ngOnDestroy() {
    localStorage.removeItem("pdfSearchTerm")

    if (this.fileUrl && this.fileUrl.startsWith("blob:")) {
      this.pdfService.revokeBlobUrl(this.fileUrl)
    }

    document.body.classList.remove("can-print")

    if (this.accessExpirationTimer) {
      this.accessExpirationTimer.unsubscribe()
    }

    if (this.accessCheckInterval) {
      this.accessCheckInterval.unsubscribe()
    }
  }

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

  toggleMetadataPanel() {
    this.isMetadataMinimized = !this.isMetadataMinimized;
  }

  toggleCommentPanel() {
    this.isCommentMinimized = !this.isCommentMinimized;
  }

  maximizePdfView() {
    this.isMetadataMinimized = true;
    this.isCommentMinimized = true;
  }

  restorePanels() {
    this.isMetadataMinimized = false;
    this.isCommentMinimized = false;
  }

  onVideoError(event: Event): void {
    this.videoError = true;
    console.error('Video failed to load:', this.fileUrl, event);
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



import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators';
import { CURRENT_API_URL } from './api-urls';

export interface BitstreamPermission {
  userId: string
  policies: BitstreamPolicy[]
  bitstreamId: string
  isAdmin?: boolean // Added isAdmin flag
}

export interface BitstreamPolicy {
  endDate: string | null
  pageStart: number
  pageEnd: number
  policyType: string
  name: string | null
  description: string | null
  action: string
  startDate: string | null
  print: boolean
  download: boolean
}

export interface TimeAccessStatus {
  hasAccess: boolean
  message: string
  validUntil: Date | null
  validFrom: Date | null
}

@Injectable({
  providedIn: "root",
})
export class BitstreamPermissionsService {
  private baseUrl = `${CURRENT_API_URL}/server/api/custom/bitstreams`

  constructor(private http: HttpClient) {}

  /**
   * Get permissions for a specific bitstream
   * @param bitstreamId The UUID of the bitstream
   * @returns Observable of BitstreamPermission
   */
  getBitstreamPermissions(bitstreamId: string): Observable<BitstreamPermission> {
    return this.http.get<BitstreamPermission>(`${this.baseUrl}/${bitstreamId}/permissions`).pipe(
      catchError((error) => {
        console.error(`Error fetching permissions for bitstream ${bitstreamId}:`, error)
        // Return an empty permission object on error
        return of({ userId: "", policies: [], bitstreamId })
      }),
    )
  }

  /**
   * Check if a bitstream has any policies (permissions) or if user is admin
   * @param bitstreamId The UUID of the bitstream
   * @returns Observable<boolean> - true if the bitstream has policies or user is admin, false otherwise
   */
  hasBitstreamPermissions(bitstreamId: string): Observable<boolean> {
    return this.getBitstreamPermissions(bitstreamId).pipe(
      map((permission) => {
        // User has permission if they are admin OR they have policies
        return permission.isAdmin === true || (permission.policies && permission.policies.length > 0)
      }),
    )
  }

  /**
   * Check if a bitstream has download permission
   * @param bitstreamId The UUID of the bitstream
   * @returns Observable<boolean> - true if download is allowed
   */
  canDownload(bitstreamId: string): Observable<boolean> {
    return this.getBitstreamPermissions(bitstreamId).pipe(
      map((permission) => {
        // Admin can always download
        if (permission.isAdmin === true) {
          return true
        }

        if (!permission.policies || permission.policies.length === 0) {
          return false
        }

        const now = new Date()
        return permission.policies.some((policy) => {
          // Check if policy allows download and is within time range
          if (!policy.download) return false

          // Check time restrictions
          const isWithinTimeRange = this.isWithinTimeRange(policy, now)
          return isWithinTimeRange
        })
      }),
    )
  }

  /**
   * Check if a bitstream has print permission
   * @param bitstreamId The UUID of the bitstream
   * @returns Observable<boolean> - true if printing is allowed
   */
  canPrint(bitstreamId: string): Observable<boolean> {
    return this.getBitstreamPermissions(bitstreamId).pipe(
      map((permission) => {
        // Admin can always print
        if (permission.isAdmin === true) {
          return true
        }

        if (!permission.policies || permission.policies.length === 0) {
          return false
        }

        const now = new Date()
        return permission.policies.some((policy) => {
          // Check if policy allows printing and is within time range
          if (!policy.print) return false

          // Check time restrictions
          const isWithinTimeRange = this.isWithinTimeRange(policy, now)
          return isWithinTimeRange
        })
      }),
    )
  }

  /**
   * Check if current time is within the policy's time range
   * @param bitstreamId The UUID of the bitstream
   * @returns Observable<TimeAccessStatus> - Access status with message
   */
  checkTimeAccess(bitstreamId: string): Observable<TimeAccessStatus> {
    return this.getBitstreamPermissions(bitstreamId).pipe(
      map((permission) => {
        // Admin always has access
        if (permission.isAdmin === true) {
          return {
            hasAccess: true,
            message: "You have admin access to this file.",
            validUntil: null,
            validFrom: null,
          }
        }

        if (!permission.policies || permission.policies.length === 0) {
          return {
            hasAccess: false,
            message: "No access policies found for this file.",
            validUntil: null,
            validFrom: null,
          }
        }

        const now = new Date()

        // Check if any policy grants access at the current time
        for (const policy of permission.policies) {
          const timeStatus = this.getTimeAccessStatus(policy, now)
          if (timeStatus.hasAccess) {
            return timeStatus
          }
        }

        // If we get here, no policy grants access at the current time
        // Find the next upcoming access window if any
        let nextAccessPolicy: BitstreamPolicy | null = null
        let nextAccessDate: Date | null = null

        for (const policy of permission.policies) {
          if (!policy.startDate) continue

          const startDate = new Date(policy.startDate)
          if (startDate > now && (!nextAccessDate || startDate < nextAccessDate)) {
            nextAccessPolicy = policy
            nextAccessDate = startDate
          }
        }

        if (nextAccessPolicy && nextAccessDate) {
          return {
            hasAccess: false,
            message: `Access will be available from ${this.formatDate(nextAccessDate)}.`,
            validUntil: null,
            validFrom: nextAccessDate,
          }
        }

        // Check if access has expired
        let lastExpiredPolicy: BitstreamPolicy | null = null
        let lastExpiredDate: Date | null = null

        for (const policy of permission.policies) {
          if (!policy.endDate) continue

          const endDate = new Date(policy.endDate)
          if (endDate < now && (!lastExpiredDate || endDate > lastExpiredDate)) {
            lastExpiredPolicy = policy
            lastExpiredDate = endDate
          }
        }

        if (lastExpiredPolicy && lastExpiredDate) {
          return {
            hasAccess: false,
            message: `Access expired on ${this.formatDate(lastExpiredDate)}.`,
            validUntil: lastExpiredDate,
            validFrom: null,
          }
        }

        return {
          hasAccess: false,
          message: "You do not have access to view this file at this time.",
          validUntil: null,
          validFrom: null,
        }
      }),
    )
  }

  /**
   * Get detailed time access status for a specific policy
   */
  private getTimeAccessStatus(policy: BitstreamPolicy, now: Date): TimeAccessStatus {
    const isWithinTimeRange = this.isWithinTimeRange(policy, now)

    if (isWithinTimeRange) {
      let message = "You have access to view this file"
      let validUntil: Date | null = null

      if (policy.endDate) {
        const endDate = new Date(policy.endDate)
        validUntil = endDate

        // Calculate time remaining
        const timeRemaining = this.getTimeRemainingText(now, endDate)
        message += ` until ${this.formatDate(endDate)} (${timeRemaining})`
      }

      return {
        hasAccess: true,
        message,
        validUntil,
        validFrom: policy.startDate ? new Date(policy.startDate) : null,
      }
    } else {
      // Access denied due to time restrictions
      if (policy.startDate && new Date(policy.startDate) > now) {
        const startDate = new Date(policy.startDate)
        return {
          hasAccess: false,
          message: `Access will be available from ${this.formatDate(startDate)}.`,
          validUntil: null,
          validFrom: startDate,
        }
      } else if (policy.endDate && new Date(policy.endDate) < now) {
        const endDate = new Date(policy.endDate)
        return {
          hasAccess: false,
          message: `Access expired on ${this.formatDate(endDate)}.`,
          validUntil: endDate,
          validFrom: null,
        }
      }

      return {
        hasAccess: false,
        message: "You do not have access to view this file at this time.",
        validUntil: null,
        validFrom: null,
      }
    }
  }

  /**
   * Check if current time is within the policy's time range
   */
  private isWithinTimeRange(policy: BitstreamPolicy, now: Date): boolean {
    // Check start date if it exists
    if (policy.startDate) {
      const startDate = new Date(policy.startDate)
      if (now < startDate) {
        return false
      }
    }

    // Check end date if it exists
    if (policy.endDate) {
      const endDate = new Date(policy.endDate)
      if (now > endDate) {
        return false
      }
    }

    return true
  }

  /**
   * Format date for display
   */
  private formatDate(date: Date): string {
    return date.toLocaleString("en-US", {
      year: "numeric",
      month: "short",
      day: "numeric",
      hour: "2-digit",
      minute: "2-digit",
    })
  }

  /**
   * Get human-readable time remaining text
   */
  private getTimeRemainingText(now: Date, endDate: Date): string {
    const diffMs = endDate.getTime() - now.getTime()
    const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24))
    const diffHours = Math.floor((diffMs % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
    const diffMinutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60))

    if (diffDays > 0) {
      return `${diffDays} day${diffDays !== 1 ? "s" : ""} ${diffHours} hour${diffHours !== 1 ? "s" : ""} remaining`
    } else if (diffHours > 0) {
      return `${diffHours} hour${diffHours !== 1 ? "s" : ""} ${diffMinutes} minute${diffMinutes !== 1 ? "s" : ""} remaining`
    } else {
      return `${diffMinutes} minute${diffMinutes !== 1 ? "s" : ""} remaining`
    }
  }
}


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
   * Creates a blob URL from a blob object
   * @param blob The blob to create a URL for
   * @returns The created blob URL
   */
  createBlobUrl(blob: Blob): string {
    return URL.createObjectURL(blob);
  }


    encryptBitstream(bitstreamId: string): Observable<Blob> {
    const url = `${CURRENT_API_URL}/server/api/diracai/encrypt-bitstream`;
    return this.http.post(url, { bitstreamId }, {
      responseType: 'blob',
      withCredentials: true
    }).pipe(
      catchError(error => {
        console.error('Encryption API error:', error);
        return throwError(() => new Error('Failed to encrypt file.'));
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

