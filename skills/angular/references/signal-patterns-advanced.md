# Signal Advanced Patterns

## Debounced Search

```typescript
import { signal, computed, inject } from '@angular/core';
import { toObservable, toSignal } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

export class SearchComponent {
  private searchService = inject(SearchService);

  protected $query = signal('');

  private $results$ = toObservable(this.$query).pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap(query => query ? this.searchService.search(query) : of([]))
  );

  protected $results = toSignal(this.$results$, { initialValue: [] });
}
```

---

## Optimistic Updates

```typescript
export class TodoListComponent {
  private todoService = inject(TodoService);
  protected $todos = signal<Todo[]>([]);

  protected async toggleComplete(todo: Todo): Promise<void> {

    const updated = { ...todo, completed: !todo.completed };

    // Optimistic update
    this.$todos.update(todos => todos.map(t => t._id === todo._id ? updated : t));

    try {
      await firstValueFrom(this.todoService.update(todo._id, { completed: updated.completed }));
    } catch {
      // Rollback on failure
      this.$todos.update(todos => todos.map(t => t._id === todo._id ? todo : t));
    }
  }
}
```

---

## Form State with Signals

```typescript
export class FilterComponent {
  protected $search = signal('');
  protected $category = signal<string | null>(null);
  protected $sortBy = signal<'name' | 'date'>('name');

  protected $activeFilters = computed(() => {
    const filters: Record<string, unknown> = {};
    const search = this.$search();
    const category = this.$category();
    if (search) filters['search'] = search;
    if (category) filters['category'] = category;
    filters['sortBy'] = this.$sortBy();
    return filters;
  });

  protected $hasFilters = computed(() => !!this.$search() || !!this.$category());

  protected clearFilters(): void {

    this.$search.set('');
    this.$category.set(null);
    this.$sortBy.set('name');
  }
}
```

---

## Signal Debugging

```typescript
// Debug effect - logs all signal reads
constructor() {
  effect(() => {
    console.log('State:', {
      items: this.$items(),
      loading: this.$loading(),
      selected: this.$selectedId()
    });
  });
}

// Trace specific signal changes
effect(() => {
  const value = this.$criticalValue();
  console.trace('criticalValue changed to:', value);
});
```

---

## Selector Pattern (NgRx-style)

```typescript
@Injectable({ providedIn: 'root' })
export class AppState {
  private readonly _$state = signal<State>(initialState);

  // Selectors - only recompute when their slice changes
  public readonly $users = computed(() => this._$state().users);
  public readonly $activeUsers = computed(() => this.$users().filter(u => u.isActive));
  public readonly $userCount = computed(() => this.$users().length);

  public updateUsers(users: User[]): void {

    this._$state.update(s => ({ ...s, users }));
  }
}
```
