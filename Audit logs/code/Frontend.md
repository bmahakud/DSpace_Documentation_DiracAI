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


import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AdminAuditRoutingModule } from './admin-audit-routing.module';
import { AuditUserListComponent } from './audit-user-list.component';
import { AuditUserDetailsComponent } from './audit-user-details.component';
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [
    AuditUserListComponent,
    AuditUserDetailsComponent
  ],
  imports: [
    CommonModule,
    FormsModule,
    AdminAuditRoutingModule
  ]
})
export class AdminAuditModule {}

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AuditUserListComponent } from './audit-user-list.component';
import { AuditUserDetailsComponent } from './audit-user-details.component';

const routes: Routes = [
  { path: '', component: AuditUserListComponent },
  { path: 'user/:userId', component: AuditUserDetailsComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AdminAuditRoutingModule {}

import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from '../core/serachpage/api-urls';

@Injectable({
  providedIn: 'root'
})
export class AdminAuditService {
  private  = '/api/audit';

  constructor(private http: HttpClient) {}

  getAllUsers(): Observable<any> {
      return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/audit/users`);
  }

  getAuditLogsByUser(userId: string): Observable<any> {
    const params = new HttpParams().set('userId', userId);
    return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/audit/user`, { params });
  }
}

<div class="user-details-container">

    <div class="summary-box">
      <p><strong>Devices Used:</strong></p>
      <ul>
        <li *ngFor="let device of uniqueDevices">{{ device }}</li>
      </ul>
    </div>
  
    <h3>Action Timeline</h3>
    <table>
      <thead>
        <tr>
          <th>Action</th>
          <th>Timestamp</th>
          <th>Object</th>
          <th>IP</th>
          <th>Device</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let log of userLogs">
          <td>{{ log.action }}</td>
          <td>{{ log.timestamp | date:'short' }}</td>
          <td>{{ log.objectId || '-' }}</td>
          <td>{{ log.ipAddress || '-' }}</td>
          <td>{{ log.userAgent || '-' }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  


.user-details-container {
    padding: 24px;
  
    .summary-box {
      background: #f9f9f9;
      padding: 12px;
      margin-bottom: 20px;
      border-radius: 4px;
    }
  
    table {
      width: 100%;
      border-collapse: collapse;
  
      th, td {
        padding: 10px;
        border: 1px solid #ccc;
      }
  
      tr:hover {
        background: #f5f5f5;
      }
    }
  }
  

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { AdminAuditService } from './admin-audit.service';

interface UserActionLog {
  action: string;
  timestamp: string;
  ipAddress: string;
  userAgent: string;
  objectId: string | null;
}

@Component({
  selector: 'app-audit-user-details',
  templateUrl: './audit-user-details.component.html',
  styleUrls: ['./audit-user-details.component.scss']
})
export class AuditUserDetailsComponent implements OnInit {
  userId: string = '';
  userLogs: UserActionLog[] = [];
  uniqueDevices: string[] = [];

  constructor(
    private route: ActivatedRoute,
    private auditService: AdminAuditService
  ) {}

  ngOnInit(): void {
    const paramId = this.route.snapshot.paramMap.get('userId');
    if (paramId) {
      this.userId = paramId;
      this.fetchUserLogs(this.userId);
    } else {
      console.error('User ID param is missing');
    }
  }

  fetchUserLogs(userId: string): void {
    this.auditService.getAuditLogsByUser(userId).subscribe({
      next: (logs: UserActionLog[]) => {
        this.userLogs = logs;
        this.uniqueDevices = [...new Set(logs.map(log => log.userAgent))];
      },
      error: (err) => {
        console.error('Error fetching audit logs:', err);
      }
    });
  }
}

<div class="user-list-container">
    <h2>User Audit Log Summary</h2>
    <table>
      <thead>
        <tr>
          <th>Email</th>
          <th>Last Login</th>
          <th>Last Logout</th>
          <th>Duration</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let user of users" (click)="goToDetails(user.userId)">
          <td><a>{{ user.email }}</a></td>
          <td>{{ user.loginTime | date:'short' }}</td>
          <td>{{ user.logoutTime | date:'short' }}</td>
          <td>{{ user.duration }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  

.user-list-container {
    padding: 24px;
  
    table {
      width: 100%;
      border-collapse: collapse;
      cursor: pointer;
  
      th, td {
        padding: 12px;
        border: 1px solid #ccc;
        text-align: left;
      }
  
      tr:hover {
        background-color: #f1f1f1;
      }
  
      a {
        text-decoration: underline;
        color: #007bff;
      }
    }
  }
  


import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { AdminAuditService } from './admin-audit.service'; // import the service

@Component({
  selector: 'app-audit-user-list',
  templateUrl: './audit-user-list.component.html',
  styleUrls: ['./audit-user-list.component.scss']
})
export class AuditUserListComponent implements OnInit {
  users: any[] = [];

  constructor(
    private router: Router,
    private auditService: AdminAuditService // inject the service
  ) {}

  ngOnInit(): void {
    this.auditService.getAllUsers().subscribe({
      next: (response) => {
        this.users = response || [];
      },
      error: (err) => {
        console.error('Error fetching users:', err);
      }
    });
  }

  goToDetails(userId: string): void {
    this.router.navigate(['/user-audit/user', userId]);
  }
}





