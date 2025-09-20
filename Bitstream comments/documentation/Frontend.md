

# 📄 Bitstream Comment Section (Frontend)

---

## 1. 📂 File Location

| File                                   | Location                                                                 |
| -------------------------------------- | ------------------------------------------------------------------------ |
| `bitstream-comment.service.ts`         | `src/app/core/serachpage/bitstream-comment.service.ts`                   |
| `viewer.component.ts` (comments logic) | `src/app/item-page/field-components/view-file-pdf/viewer.component.ts`   |
| `viewer.component.html` (comments UI)  | `src/app/item-page/field-components/view-file-pdf/viewer.component.html` |

---

## 2. 📝 Code

### `bitstream-comment.service.ts` (service for API calls)

```ts
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
```

---

### `viewer.component.ts` (comment integration)

```ts
comments: BitstreamComment[] = [];
newComment: string = '';

loadComments(): void {
  this.commentService.getComments(this.bitstreamUuid).subscribe(comments => {
    this.comments = comments;
  });
}

addComment(): void {
  if (this.newComment.trim() === '') return;
  const comment: BitstreamComment = { bitstreamId: this.bitstreamUuid, comment: this.newComment };
  this.commentService.addComment(comment).subscribe(() => {
    this.newComment = '';
    this.loadComments();
  });
}

confirmDeleteComment(commentId: number): void {
  if (confirm('Are you sure you want to delete this comment?')) {
    this.commentService.deleteComment(commentId).subscribe(() => this.loadComments());
  }
}
```

---

### `viewer.component.html` (UI snippet)

```html
<div class="comments-panel">
  <h3>Comments</h3>

  <div *ngFor="let c of comments" class="comment-item">
    <span>{{ c.text }}</span>
    <button *ngIf="isAdmin" (click)="confirmDeleteComment(c.id)">Delete</button>
  </div>

  <div *ngIf="isAdmin" class="comment-input">
    <textarea [(ngModel)]="newComment" placeholder="Add a comment"></textarea>
    <button (click)="addComment()">Add Comment</button>
  </div>
</div>
```

---

## 3. ✅ Features

* 📥 **Fetch comments** for the selected bitstream.
* ✍️ **Add new comments** (only admins).
* 🗑 **Delete comments** (with confirmation, only admins).
* 📝 **Update comments** (optional, via service).
* 🔒 Uses `withCredentials: true` → ensures authentication session is sent.
* 🎨 UI shows list of comments + admin-only input box.

---

## 4. 🔍 Feature-wise Explanation

* **Service (API layer)**:
  Handles CRUD operations (get, add, update, delete) with backend.

* **ViewerComponent logic**:

  * `loadComments()` → fetches and sets comments array.
  * `addComment()` → sends new comment to backend, refreshes list.
  * `confirmDeleteComment()` → deletes comment after confirmation popup.

* **UI (`viewer.component.html`)**:

  * Displays list of comments.
  * Shows delete button only if user is admin.
  * Shows input box for new comments only if user is admin.

---

## 5. 📖 Line-by-Line Explanation

### Service (`bitstream-comment.service.ts`)

```ts
private baseUrl = `${CURRENT_API_URL}/server/api/bitstream/comment`;
```

➡ Defines the base endpoint for comment APIs.

```ts
getComments(bitstreamId: string)
```

➡ Calls `GET /bitstream/{id}` to fetch comments.

```ts
addComment(comment: BitstreamComment)
```

➡ Calls `POST /comment` to add a new one.

```ts
updateComment(id: number, newText: string)
```

➡ Calls `PUT /comment/{id}` with plain text body.

```ts
deleteComment(id: number)
```

➡ Calls `DELETE /comment/{id}`.

---

### Component (`viewer.component.ts`)

```ts
comments: BitstreamComment[] = [];
newComment: string = '';
```

➡ Holds current comments & new input text.

```ts
loadComments()
```

➡ Refreshes comment list from backend.

```ts
addComment()
```

➡ Posts new comment → clears input → reloads comments.

```ts
confirmDeleteComment()
```

➡ Confirms → deletes comment → refreshes list.

---

### UI (`viewer.component.html`)

```html
<div *ngFor="let c of comments" class="comment-item">
  <span>{{ c.text }}</span>
  <button *ngIf="isAdmin" (click)="confirmDeleteComment(c.id)">Delete</button>
</div>
```

➡ Iterates comments.
➡ Shows Delete button only for admins.

```html
<div *ngIf="isAdmin" class="comment-input">
  <textarea [(ngModel)]="newComment"></textarea>
  <button (click)="addComment()">Add Comment</button>
</div>
```

➡ Textarea + button only visible for admins.
➡ `[(ngModel)]` binds textarea → `newComment`.

---

✅ In summary:

* Service = API calls
* Component = business logic
* Template = UI rendering

---
