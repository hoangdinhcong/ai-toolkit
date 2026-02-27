# Data Fetching Patterns

## rxResource - Reactive Data Loading

Use `rxResource` when data depends on reactive params (signals/inputs) and needs loading/error state.

### Basic Pattern

```typescript
import { computed, ResourceRef } from '@angular/core';
import { rxResource } from '@angular/core/rxjs-interop';

export class DetailComponent {
  protected readonly $id = injectParams('id');

  protected readonly $resource: ResourceRef<Item | null> = rxResource({
    params: () => ({ id: this.$id() }),
    stream: ({ params: { id } }) => {

      if (!id) return of(null);
      return this.entityDataService.getById<Item>(ActivityEntityName.Item, id).pipe(
        map(response => response?.item ?? null)
      );
    }
  });

  // Expose as signals
  protected readonly $item = computed(() => this.$resource.value() ?? null);
  protected readonly $isLoading = computed(() => this.$resource.isLoading());
  protected readonly $error = computed(() => this.$resource.error());
}
```

### Reload on External Events

```typescript
constructor() {
  this.socketService.events$.pipe(
    filter(id => id === this.$id()),
    takeUntilDestroyed()
  ).subscribe(() => {
    this.$resource.reload();
  });
}
```

### Error Handling

```typescript
protected $resource = rxResource({
  params: () => ({ id: this.$id() }),
  stream: ({ params: { id } }) => {

    return this.http.get<Item>(`/api/items/${id}`).pipe(
      catchError(() => of(null))
    );
  }
});
```

### Key Rules

- Return `of(null)` for invalid/empty params - don't throw
- Use `computed()` to expose resource values as clean signals
- Call `.reload()` for manual refresh
- Params are reactive - changing signal values auto-refetches

---

## takeUntilDestroyed - Subscription Cleanup

Auto-unsubscribe when component is destroyed. Call in constructor or field initializer (injection context required).

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class MyComponent {
  constructor() {
    this.service.events$.pipe(
      takeUntilDestroyed()
    ).subscribe(event => {
      this.$data.set(event);
    });
  }
}
```

### Outside Injection Context

Pass `DestroyRef` when called outside constructor:

```typescript
private destroyRef = inject(DestroyRef);

ngOnInit(): void {

  this.service.data$.pipe(
    takeUntilDestroyed(this.destroyRef)
  ).subscribe(data => this.$data.set(data));
}
```

### Key Rules

- Must be called in injection context (constructor) or pass `DestroyRef`
- Place as **last** operator in the pipe chain
- `toSignal()` has implicit `takeUntilDestroyed` - don't double up
- Prefer `rxResource` for data loading; use `takeUntilDestroyed` for event streams

---

## Service with API Integration

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class ItemService {
  private http = inject(HttpClient);
  private api = 'api/items';

  public getAll(): Observable<Item[]> {

    return this.http.get<Item[]>(this.api);
  }

  public getById(id: string): Observable<Item> {

    return this.http.get<Item>(`${this.api}/${id}`);
  }

  public create(data: Partial<Item>): Observable<Item> {

    return this.http.post<Item>(this.api, data);
  }

  public update(id: string, data: Partial<Item>): Observable<Item> {

    return this.http.patch<Item>(`${this.api}/${id}`, data);
  }
}
```

## Decision Guide

| Scenario | Pattern |
|----------|---------|
| Data depends on reactive params, needs loading state | `rxResource` |
| One-shot fetch in async method | `firstValueFrom` |
| Sequential async operations (dialog -> command) | `firstValueFrom` |
| Long-lived event stream (socket, form changes) | `.subscribe()` + `takeUntilDestroyed()` |
| Convert observable to signal | `toSignal()` (implicit cleanup) |
