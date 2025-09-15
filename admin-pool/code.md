import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AdminPoolComponent } from './admin-pool.component';
import { AdminPoolRoutingModule } from './admin-pool-routing.module';
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [AdminPoolComponent],
  imports: [
    CommonModule,
    FormsModule,
    AdminPoolRoutingModule,
  ],
})
export class AdminPoolModule {}




import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';
import { CURRENT_API_URL } from '../core/serachpage/api-urls';

@Injectable({
  providedIn: 'root',
})
export class AdminPoolService {
  constructor(private http: HttpClient) {}

  fetchBatches(status: string): Observable<any[]> {
    return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/bulk-upload/status/${status}`);
  }

  getBatchFiles(batchId: string): Observable<any> {
    return this.http.get<any>(`${CURRENT_API_URL}/server/api/bulk-upload/${batchId}`);
  }

  approve(uuid: string, collectionUuid: string): Observable<any> {
    const zipFilename = `${uuid}.zip`;

    const properties = [
      { name: '--add' },
      { name: '--zip', value: zipFilename },
      { name: '--collection', value: collectionUuid }
    ];
  
    const body = new URLSearchParams();

    body.set('properties', JSON.stringify(properties));
    console.log(body.toString());

    console.log(collectionUuid);
    const headers = new HttpHeaders({
      'Content-Type': 'application/x-www-form-urlencoded'
    });
  
    return this.http.post(
      `${CURRENT_API_URL}/server/api/bulk-upload/approve/${uuid}`,
      body.toString(),
      { headers }
    );
  }
  
  reject(uuid: string): Observable<any> {
    return this.http.post(`${CURRENT_API_URL}/server/api/bulk-upload/reject/${uuid}`, {});
  }

  getPooledTasks(): Observable<any[]> {
    return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/bulk-upload/pooled`);
  }

  getAcceptedSubmissions(): Observable<any[]> {
    return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/bulk-upload/status/APPROVED`);
  }

  getRejectedSubmissions(): Observable<any[]> {
    return this.http.get<any[]>(`${CURRENT_API_URL}/server/api/bulk-upload/status/REJECTED`);
  }
  
  
} 





import { Component, ChangeDetectorRef } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { OnInit } from '@angular/core';
import { CURRENT_API_URL } from '../core/serachpage/api-urls'; 
import { AdminPoolService } from './admin-service';

@Component({
    selector: 'app-admin-pool',
    templateUrl: './admin-pool.component.html',
    styleUrls: ['./admin-pool.component.scss'],
})
export class AdminPoolComponent implements OnInit {

    constructor(private adminPoolService: AdminPoolService, private cdr: ChangeDetectorRef) { }
    claimedTasks = [];
    pooledTasks = [];
    rejectedTasks = [];

    ngOnInit() {
        this.fetchClaimedTasks();
        this.fetchPooledTasks();
        this.fetchRejectedTasks();
        this.fetchAcceptedSubmissions();
    }

    fetchClaimedTasks() {
        this.adminPoolService.fetchBatches('CLAIMED').subscribe(
            (res) => {
                this.claimedTasks = res;
                this.cdr.markForCheck();
            },
            (err) => {
                console.error('Failed to fetch CLAIMED tasks', err);
                this.cdr.markForCheck();
            }
        );
    }

    fetchPooledTasks() {
        this.adminPoolService.getPooledTasks().subscribe(
            (res) => {
                this.pooledTasks = res;
                this.cdr.markForCheck();
            },
            (err) => {
                console.error('Failed to fetch pooled tasks', err);
                this.cdr.markForCheck();
            }
        );
    }

    fetchRejectedTasks() {
        this.adminPoolService.getRejectedSubmissions().subscribe(
            (res) => {
                this.rejectedTasks = res;
                this.cdr.markForCheck();
            },
            (err) => {
                console.error('Failed to fetch rejected tasks', err);
                this.cdr.markForCheck();
            }
        );
    }

    fetchAcceptedSubmissions() {
        this.adminPoolService.getAcceptedSubmissions().subscribe(
            (res) => {
                this.acceptedSubmissions = res;
                this.cdr.markForCheck();
            },
            (err) => {
                console.error('Failed to fetch accepted submissions', err);
                this.acceptedSubmissions = [];
                this.cdr.markForCheck();
            }
        );
    }

    selectedBatch: any = null;

    dummyFiles: any[] = [];  
    acceptedSubmissions: any[] = []; 

    actionUUID :any = null;
    collectionUuid :any = null; 
    reviewLoading: boolean = false;

    openReviewDialog(batch: any) {
        this.selectedBatch = batch;
        this.reviewLoading = true;
        const batchId = batch.bulkFileId;
        this.collectionUuid = batch.collection.collectionId;

        this.adminPoolService.getBatchFiles(batchId).subscribe(
            (res) => {

                this.actionUUID = res.requestId;
                this.dummyFiles = res.items.map(item => {
                    if (typeof item.metadata === 'string') {
                        try {
                            item.metadata = JSON.parse(item.metadata);
                        } catch (e) {
                            console.error("Failed to parse metadata for item:", item, e);
                            item.metadata = {}; // Fallback to empty object
                        }
                    }
                    return item;
                });
                this.reviewLoading = false;
                this.cdr.markForCheck();
                console.log("testing in the dialogue box",this.collectionUuid);

            },
            (err) => {
                console.error(`Failed to fetch files for batch ${batchId}`, err);
                this.dummyFiles = [];
                this.reviewLoading = false;
                this.cdr.markForCheck();
            }
        );
    }
    
    
    
    approve(uuid: string) {
      
        this.adminPoolService.approve(this.actionUUID, this.collectionUuid).subscribe(() => {
            console.log(this.collectionUuid);

          alert('✅ Approved successfully.');
          this.fetchClaimedTasks();
          this.fetchPooledTasks();
          this.fetchRejectedTasks();
          this.fetchAcceptedSubmissions();
          this.cancelReview();
        });
      }

    reject(uuid: string) {
        this.adminPoolService.reject(this.actionUUID).subscribe(() => {
            alert('Rejected successfully.');
            this.fetchClaimedTasks();
            this.fetchPooledTasks();
            this.fetchRejectedTasks();
            this.fetchAcceptedSubmissions();
            this.cancelReview();
            
        });
    }


    cancelReview() {
        this.selectedBatch = null;
        this.reviewLoading = false;
    }

    getBatchInfo() {
        alert("Fetching batch info...");
        // Placeholder for API integration
    }

    showAccepted = false;
    viewingAcceptedBatch: any = null;

    viewAcceptedSubmissions() {
        this.adminPoolService.getAcceptedSubmissions().subscribe(
            (res) => {
                this.acceptedSubmissions = res;
                this.showAccepted = true;
                this.viewingAcceptedBatch = null;
                this.cdr.markForCheck();
            },
            (err) => {
                console.error('Failed to fetch accepted submissions', err);
                this.acceptedSubmissions = [];
                this.showAccepted = true;
                this.viewingAcceptedBatch = null;
                this.cdr.markForCheck();
            }
        );
    }

    viewFiles(batch: any) {
        this.viewingAcceptedBatch = batch;
    }

    closeAcceptedView() {
        this.showAccepted = false;
        this.viewingAcceptedBatch = null;
    }
}




.admin-pool {
    padding: 20px;

    h2 {
        margin-top: 30px;
        margin-bottom: 10px;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 30px;

        th,
        td {
            border: 1px solid #ccc;
            padding: 10px;
            text-align: left;
        }

        th {
            background-color: #f8f8f8;
        }

        button {
            padding: 6px 12px;
            background-color: #007bff;
            color: white;
            border: none;
            cursor: pointer;
        }

        button:hover {
            background-color: #0056b3;
        }
    }

    .top-actions {
        margin-bottom: 20px;

        button {
            margin-right: 10px;
            padding: 6px 12px;
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;

            &:hover {
                background-color: #218838;
            }
        }
    }

    .review-box {
        margin-top: 40px;
        padding: 20px;
        border: 1px solid #ccc;
        background: #fdfdfd;

        .actions {
            margin-top: 20px;

            button {
                margin-right: 10px;
                padding: 6px 12px;
                background-color: #007bff;
                color: white;
                border: none;

                &:hover {
                    background-color: #0056b3;
                }
            }
        }
    }
}



.accepted-submissions {
    margin-top: 40px;
    padding: 20px;
    border: 1px solid #ccc;
    background: #f4f8fa;

    table {
        margin-top: 10px;
        width: 100%;
        border-collapse: collapse;

        th,
        td {
            padding: 10px;
            border: 1px solid #ddd;
        }

        th {
            background-color: #e6eef5;
        }

        button {
            padding: 6px 12px;
            background-color: #17a2b8;
            color: white;
            border: none;

            &:hover {
                background-color: #138496;
            }
        }
    }

    ul {
        list-style-type: disc;
        margin-left: 20px;
    }

    h3 {
        margin-top: 20px;
    }
}





<div class="admin-pool">
    <!-- Top Buttons -->
    <div class="top-actions">
        <button (click)="viewAcceptedSubmissions()">View Accepted Submissions</button>
        <button (click)="getBatchInfo()">Get Batch Info</button>
    </div>

    <!-- Rejected Batches -->
    <h2>Rejected Batches</h2>
    <table *ngIf="rejectedTasks.length > 0; else noRejected">
        <thead>
            <tr>
                <th>Filename</th>
                <th>Uploader</th>
                <th>Collection</th>
                <th>reviewer</th>
                <th>reviewed Date</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let task of rejectedTasks">
                <td>{{ task.fileName }}</td>
                <td>{{ task.uploader.userName }}</td>
                <td>{{ task.collection.collectionName }}</td>
                <td>{{ task.reviewer.userName }}</td>
                <td>{{ task.reviewer.date | date:'short' }}</td>
            </tr>
        </tbody>
    </table>
    <ng-template #noRejected>
        <p>No rejected batches available.</p>
    </ng-template>

    <!-- Claimed Tasks -->
    <h2>Claimed Tasks</h2>
    <table *ngIf="claimedTasks.length > 0; else noClaimed">
        <thead>
            <tr>
                <th>Filename</th>
                <th>Uploader</th>
                <th>Collection</th>
                <th>Status</th>
                <th>Uploaded Date</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let task of claimedTasks">
                <td>{{ task.fileName }}</td>
                <td>{{ task.uploader.userName }}</td>
                <td>{{ task.collection.collectionName }}</td>
                <td>{{ task.status }}</td>
                <td>{{ task.uploader.date | date:'short' }}</td>
                <td>
                    <button (click)="openReviewDialog(task)">Perform Task</button>
                </td>
            </tr>
        </tbody>
    </table>
    <ng-template #noClaimed>
        <p>No claimed tasks available.</p>
    </ng-template>


    <!-- Pooled Tasks -->
    <h2>Pooled Tasks</h2>
    <table>
        <thead>
            <tr>
                <th>Filename</th>
                <th>Uploader</th>
                <th>Collection</th>
                <th>Status</th>
                <th>reviewer</th>
                <th>Uploaded Date</th>
                <th>reviewed Date</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let task of pooledTasks">
                <td>{{ task.fileName }}</td>
                <td>{{ task.uploader.userName }}</td>
                <td>{{ task.collection.collectionName }}</td>
                <td>{{ task.status }}</td>
                <td>{{ task.reviewer.userName }}</td>
                <td>{{ task.uploader.date | date:'short' }}</td>
                <td>{{ task.reviewer.date | date:'short' }}</td>
                <td><button (click)="openReviewDialog(task)">Show Task</button></td>
            </tr>
        </tbody>
    </table>

    <!-- Review Batch Section -->
    <div *ngIf="selectedBatch" class="review-box">
        <h3>Reviewing Batch: {{ selectedBatch.batch }}</h3>
        <div *ngIf="reviewLoading" class="loading-spinner">
            Loading files, please wait...
        </div>
        <table *ngIf="!reviewLoading">
            <thead>
                <tr>
                    <th>Case Type</th>
                    <th>Case No</th>
                    <th>Year</th>
                    <th>Disposal Date</th>
                    <th>Petitioner</th>
                    <th>Respondent</th>
                    <th>Item Folder</th>
                </tr>
            </thead>
            <tbody>
                <tr *ngFor="let file of dummyFiles">
                    <td>{{ file.metadata?.['dc.casetype'] || '-' }}</td>
                    <td>{{ file.metadata?.['dc.title'] || '-' }}</td>
                    <td>{{ file.metadata?.['dc.caseyear'] || '-' }}</td>
                    <td>{{ file.metadata?.['dc.date.disposal'] || '-' }}</td>
                    <td>{{ file.metadata?.['dc.pname'] || '-' }}</td>
                    <td>{{ file.metadata?.['dc.rname'] || '-' }}</td>
                    <td>{{ file.itemFolder }}</td>
                </tr>
            </tbody>
        </table>
        <div class="actions" *ngIf="!reviewLoading">
            <button (click)="approve(selectedBatch.requestId)">Approve</button>
            <button (click)="reject(selectedBatch.requestId)">Reject</button>
            <button (click)="cancelReview()">Cancel</button>
        </div>
    </div>

   <!-- Accepted Submissions Section -->
   <div *ngIf="showAccepted" class="accepted-submissions">
    <h2>Accepted Submissions</h2>
    <table *ngIf="!viewingAcceptedBatch">
        <thead>
            <tr>
                <th>Filename</th>
                <th>Uploader</th>
                <th>Collection</th>
                <th>Reviewer</th>
                <th>Status</th>
                <th>Uploaded Date</th>
                <th>Reviewed Date</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let accepted of acceptedSubmissions">
                <td>{{ accepted.fileName }}</td>
                <td>{{ accepted.uploader.userName }}</td>
                <td>{{ accepted.collection.collectionName }}</td>
                <td>{{ accepted.reviewer.userName }}</td>
                <td>{{ accepted.status }}</td>
                <td>{{ accepted.uploader.date | date:'short' }}</td>
                <td>{{ accepted.reviewer.date | date:'short' }}</td>
                <td><button (click)="viewFiles(accepted)">View Files</button></td>
            </tr>
        </tbody>
    </table>
    <div *ngIf="viewingAcceptedBatch">
        <h3>Files in Batch: {{ viewingAcceptedBatch.batch }}</h3>
        <ul>
            <li *ngFor="let file of viewingAcceptedBatch.files">{{ file }}</li>
        </ul>
        <button (click)="viewingAcceptedBatch = null">Back</button>
    </div>
    <button (click)="closeAcceptedView()" style="margin-top: 15px;">Close Accepted Submissions</button>
</div>

</div>





import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { AdminPoolComponent } from './admin-pool.component';

const routes: Routes = [
  {
    path: '',
    component: AdminPoolComponent,
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AdminPoolRoutingModule {}


