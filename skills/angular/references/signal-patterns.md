# Signal Patterns

## Core Signals

### signal() - Writable State

```typescript
// Typed
protected $value = signal<string>('initial');
protected $items = signal<Item[]>([]);

// Set (replace)
this.$value.set('new value');

// Update (from previous)
this.$items.update(prev => [...prev, newItem]);
```

### computed() - Derived State

```typescript
protected $fullName = computed(() => `${this.$firstName()} ${this.$lastName()}`);
protected $isValid = computed(() => this.$items().length > 0 && !this.$error());
protected $filtered = computed(() => this.$items().filter(i => i.status === this.$filter()));
```

Computed signals are lazy (only recalculate when read) and cached (don't recalculate if dependencies haven't changed).

### linkedSignal() - Reactive Watcher

Resets to a new value whenever dependencies change:

```typescript
// Recompute when input changes
protected $selectedTab = linkedSignal(() => {
  const tabs = this.$tabs();
  return tabs[0] ?? null;
});

// With previous value access
protected $page = linkedSignal({
  source: this.$filter,
  computation: (filter, previous) => previous?.source === filter ? previous.value : 1
});
```

### effect() - Side Effects

```typescript
constructor() {
  // Basic effect
  effect(() => {
    const viewId = this.$viewId();
    if (!viewId) return;
    this.loadData(viewId);
  });

  // Untracked reads (won't re-trigger effect)
  effect(() => {
    const primary = this.$primary();
    untracked(() => {
      const secondary = this.$secondary();
      this.doSomething(primary, secondary);
    });
  });

  // With cleanup
  effect((onCleanup) => {
    const sub = this.service.stream$.subscribe(d => this.$data.set(d));
    onCleanup(() => sub.unsubscribe());
  });
}
```

## Signal Equality

Prevent unnecessary updates for object/array signals:

```typescript
protected $filters = signal<EntityViewFilter[]>([], {
  equal: (a, b) => JSON.stringify(a) === JSON.stringify(b)
});

protected $config = signal<Config>(defaultConfig, {
  equal: (a, b) => a.id === b.id && a.version === b.version
});
```

## RxJS Interop

### toSignal() - Observable to Signal

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

// With initial value
protected $routeId = toSignal(this.route.paramMap.pipe(map(p => p.get('id'))), { initialValue: null });

// Required initial value for synchronous access
protected $currentUrl = toSignal(this.router.events.pipe(map(() => this.router.url)), { initialValue: this.router.url });
```

`toSignal()` automatically handles cleanup (implicit `takeUntilDestroyed`).

### toObservable() - Signal to Observable

```typescript
import { toObservable } from '@angular/core/rxjs-interop';

// React to signal changes as observable stream
toObservable(this.$searchTerm).pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.searchService.search(term)),
  takeUntilDestroyed()
).subscribe(results => this.$results.set(results));
```

## Component State Pattern

For components with multiple related state fields:

```typescript
export class DashboardComponent {
  // Group related state
  protected $items = signal<Item[]>([]);
  protected $loading = signal(false);
  protected $error = signal<string>(null);
  protected $selectedId = signal<string>(null);

  // Derive from state
  protected $selectedItem = computed(() => {
    const id = this.$selectedId();
    return id ? this.$items().find(i => i._id === id) ?? null : null;
  });

  protected $isEmpty = computed(() => !this.$loading() && this.$items().length === 0);
}
```

## Service State Pattern

For shared state across components:

```typescript
@Injectable({ providedIn: 'root' })
export class CartService {
  // Private writable signals
  private readonly _$items = signal<CartItem[]>([]);
  private readonly _$loading = signal(false);

  // Public read-only
  public readonly $items = this._$items.asReadonly();
  public readonly $loading = this._$loading.asReadonly();
  public readonly $total = computed(() => this._$items().reduce((sum, i) => sum + i.price * i.qty, 0));
  public readonly $count = computed(() => this._$items().length);

  public addItem(item: CartItem): void {

    this._$items.update(items => [...items, item]);
  }

  public removeItem(id: string): void {

    this._$items.update(items => items.filter(i => i._id !== id));
  }
}
```

---

For advanced patterns (debounced search, optimistic updates, form state with signals, signal debugging), see [signal-patterns-advanced.md](signal-patterns-advanced.md)
```
