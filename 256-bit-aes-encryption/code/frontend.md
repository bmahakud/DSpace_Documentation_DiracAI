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





@Component({
  selector: "app-viewer",
  templateUrl: "./viewer.component.html",
  styleUrls: ["./viewer.component.scss"],
  standalone: true,
  imports: [CommonModule, FormsModule],
  encapsulation: ViewEncapsulation.None,
})

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




import { NgModule } from "@angular/core"
import { CommonModule } from "@angular/common"
import { SafePipeModule } from "./safe-pipe.module"

@NgModule({
  declarations: [],
  imports: [CommonModule, SafePipeModule],
  exports: [],
})
export class ViewerModule {}

