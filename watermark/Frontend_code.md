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
        path: 'admin-watermark',
        component: (AdminPannelComponent as any), // import the component at the top
        canActivate: [authenticatedGuard, endUserAgreementCurrentUserGuard]
      }
    ] 
   } 
]

import { Component, OnDestroy } from '@angular/core';
import { HttpEvent, HttpEventType } from '@angular/common/http';
import { WatermarkApiService } from './admin-pannel.service';

@Component({
  selector: 'app-admin-pannel',
  templateUrl: './admin-pannel.component.html',
  styleUrls: ['./admin-pannel.component.scss']
})
export class AdminPannelComponent implements OnDestroy {
  imgUrl: string | null = null;
  progress = -1;

  // For drag-and-drop UI
  isDragOver = false;

  constructor(private api: WatermarkApiService) {
    this.refresh();
  }

  ngOnDestroy(): void {
    if (this.imgUrl) {
      URL.revokeObjectURL(this.imgUrl);
      this.imgUrl = null;
    }
  }

  refresh(): void {
    this.api.getCurrent().subscribe({
      next: (blob) => {
        if (this.imgUrl) URL.revokeObjectURL(this.imgUrl);
        this.imgUrl = (!blob || blob.size === 0) ? null : URL.createObjectURL(blob);
      },
      error: () => { this.imgUrl = null; }
    });
  }

  // Handles manual file selection
  onPick(ev: Event): void {
    const input = ev.target as HTMLInputElement;
    if (!input.files?.length) return;
    const file = input.files[0];
    this.uploadFile(file);
  }

  // ✅ DRAG-AND-DROP SUPPORT
  onDragOver(event: DragEvent) {
    event.preventDefault();
    this.isDragOver = true;
  }

  onDragLeave(event: DragEvent) {
    event.preventDefault();
    this.isDragOver = false;
  }

  onDrop(event: DragEvent) {
    event.preventDefault();
    this.isDragOver = false;
    const file = event.dataTransfer?.files[0];
    if (file) {
      this.uploadFile(file);
    }
  }

  // ✅ Shared upload logic for both file picker and drag-drop
  private uploadFile(file: File) {
    this.progress = 0;
    this.api.upload(file).subscribe({
      next: (evt: HttpEvent<any>) => {
        if (evt.type === HttpEventType.UploadProgress && evt.total) {
          this.progress = Math.round((evt.loaded / evt.total) * 100);
        }
      },
      error: (err) => {
        console.error('Upload failed', err);
        this.progress = -1;
      },
      complete: () => {
        this.progress = 100;
        this.refresh();
        setTimeout(() => (this.progress = -1), 1200);
      }
    });
  }
}

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

import { AdminPannelComponent } from './admin-pannel.component';

@NgModule({
  declarations: [AdminPannelComponent],
  imports: [
    CommonModule,
    FormsModule,
    HttpClientModule
  ],
  exports: [AdminPannelComponent] 
})
export class AdminPannelModule {}



import { Injectable } from '@angular/core';
import { HttpClient, HttpEvent, HttpRequest } from '@angular/common/http';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from '../core/serachpage/api-urls';

const BASE = `${CURRENT_API_URL}/server/api/watermark`;

@Injectable({
  providedIn: 'root'
})
export class WatermarkApiService {
  constructor(private http: HttpClient) {}

  getCurrent(): Observable<Blob> {
    return this.http.get(BASE, { responseType: 'blob', withCredentials: true });
  }

  upload(file: File): Observable<HttpEvent<any>> {
    const form = new FormData();
    form.append('file', file);
    const req = new HttpRequest('POST', BASE, form, {
      reportProgress: true,
      withCredentials: true
    });
    return this.http.request(req);
  }
}





.watermark-page {
  .watermark-preview {
    max-width: 100%;
    max-height: 300px;
    border-radius: 8px;
    border: 1px solid #ddd;
    object-fit: contain;
    padding: 8px;
    background-color: #f8f9fa;
  }

  .empty-state {
    padding: 2rem 1rem;
  }

  .dropzone {
    position: relative;
    display: block;
    width: 100%;
    min-height: 160px;
    border: 2px dashed #ccc;
    border-radius: 8px;
    background: #fafafa;
    cursor: pointer;
    text-align: center;
    transition: all 0.2s ease-in-out;

    &.drag-over {
      border-color: #0d6efd;
      background: #eaf2ff;
    }

    input[type='file'] {
      position: absolute;
      inset: 0;
      opacity: 0;
      cursor: pointer;
    }

    .dropzone-inner {
      padding: 2rem 1rem;
      i {
        font-size: 2rem;
        color: #0d6efd;
        margin-bottom: 0.5rem;
      }
      p {
        font-weight: 500;
        margin: 0;
      }
      small {
        color: #777;
      }
    }
  }
}




<div class="watermark-page container py-4">
  <h2 class="mb-4">Watermark Management</h2>

  <div class="row g-4">
    <!-- Current Watermark -->
    <div class="col-lg-6">
      <div class="card shadow-sm h-100">
        <div class="card-header bg-light fw-bold">
          Current Watermark
        </div>
        <div class="card-body d-flex flex-column align-items-center justify-content-center text-center">
          <ng-container *ngIf="imgUrl; else noWatermark">
            <img [src]="imgUrl" alt="Current Watermark" class="watermark-preview mb-3" />

          </ng-container>

          <ng-template #noWatermark>
            <div class="empty-state">
              <i class="bi bi-image" style="font-size: 2.5rem; color: #aaa;"></i>
              <p class="text-muted mt-2">No watermark uploaded yet</p>
              <small class="text-muted">Upload an image to set as the default watermark</small>
            </div>
          </ng-template>
        </div>
      </div>
    </div>

    <!-- Upload Section -->
    <div class="col-lg-6">
      <div class="card shadow-sm h-100">
        <div class="card-header bg-light fw-bold">
          Upload New Watermark
        </div>
        <div class="card-body">
          <!-- Dropzone -->
          <label
            class="dropzone"
            (dragover)="onDragOver($event)"
            (dragleave)="onDragLeave($event)"
            (drop)="onDrop($event)"
            [class.drag-over]="isDragOver"
          >
            <input type="file" accept="image/*" (change)="onPick($event)" />
            <div class="dropzone-inner">
              <i class="bi bi-cloud-arrow-up"></i>
              <p>Drag & Drop your image here</p>
              <small class="text-muted">or click to browse</small>
            </div>
          </label>

          <!-- Progress Bar -->
          <div *ngIf="progress >= 0" class="mt-3">
            <div class="progress">
              <div
                class="progress-bar progress-bar-striped progress-bar-animated"
                role="progressbar"
                [style.width.%]="progress"
              >
                {{ progress }}%
              </div>
            </div>
          </div>

          <!-- Helper Text -->
          <p class="mt-3 text-muted small">
            Accepted formats: PNG, JPG, WEBP — Recommended: Transparent PNG
          </p>
        </div>
      </div>
    </div>
  </div>
</div>

