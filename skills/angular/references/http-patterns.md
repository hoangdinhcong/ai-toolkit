# HTTP Patterns

## HttpClient Usage

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);

  public get<T>(url: string, params?: Record<string, string>): Observable<T> {

    const httpParams = params ? new HttpParams({ fromObject: params }) : undefined;
    return this.http.get<T>(url, { params: httpParams });
  }

  public post<T>(url: string, body: unknown): Observable<T> {

    return this.http.post<T>(url, body);
  }

  public patch<T>(url: string, body: unknown): Observable<T> {

    return this.http.patch<T>(url, body);
  }

  public delete<T>(url: string): Observable<T> {

    return this.http.delete<T>(url);
  }
}
```

---

## Functional Interceptors

```typescript
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';

// Auth token interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {

  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
  }
  return next(req);
};

// Error handling interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        inject(AuthService).logout();
      }
      return throwError(() => error);
    })
  );
};

// Logging interceptor
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {

  const started = Date.now();
  return next(req).pipe(
    tap(() => console.log(`${req.method} ${req.url} - ${Date.now() - started}ms`))
  );
};
```

### Register Interceptors

```typescript
// In app.config.ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';

export const appConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor]))
  ]
};
```

---

## Error Handling

```typescript
import { catchError, retry, timer } from 'rxjs';

// Basic catch
this.http.get<Data>('/api/data').pipe(
  catchError(error => {
    console.error('Request failed:', error);
    return of(null);
  })
);

// Retry with backoff
this.http.get<Data>('/api/data').pipe(
  retry({ count: 3, delay: (error, retryCount) => timer(retryCount * 1000) }),
  catchError(() => of(null))
);
```

---

## Request Options

```typescript
// With query params
this.http.get<Item[]>('/api/items', {
  params: new HttpParams().set('page', '1').set('limit', '20')
});

// With headers
this.http.post<Item>('/api/items', data, {
  headers: new HttpHeaders({ 'Content-Type': 'application/json' })
});

// Get full response (headers, status, etc.)
this.http.get<Item>('/api/items/1', { observe: 'response' }).pipe(
  map(response => ({ data: response.body, etag: response.headers.get('ETag') }))
);

// Text response
this.http.get('/api/export', { responseType: 'text' });

// Blob (file download)
this.http.get('/api/files/report.pdf', { responseType: 'blob' });
```
