# Dependency Injection Patterns

## inject() Function

Always use `inject()`, never constructor injection.

```typescript
import { inject, DestroyRef, ElementRef } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';

export class MyComponent {
  // Services
  protected readonly router = inject(Router);
  protected readonly route = inject(ActivatedRoute);
  private readonly destroyRef = inject(DestroyRef);

  // Optional injection
  protected readonly analytics = inject(AnalyticsService, { optional: true });

  // Element reference
  private readonly el = inject(ElementRef);
}
```

---

## InjectionToken

```typescript
import { InjectionToken, inject } from '@angular/core';

// Define token
export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');
export const FEATURE_FLAGS = new InjectionToken<FeatureFlags>('FEATURE_FLAGS');

// Provide in module/component
@Component({
  providers: [
    { provide: API_BASE_URL, useValue: '/api/v2' }
  ]
})

// Inject
export class ApiService {
  private baseUrl = inject(API_BASE_URL);
}
```

### Factory Token

```typescript
export const LOGGER = new InjectionToken<Logger>('LOGGER', {
  providedIn: 'root',
  factory: () => {
    const env = inject(EnvironmentService);
    return env.isProduction() ? new ProductionLogger() : new ConsoleLogger();
  }
});
```

---

## providedIn Options

```typescript
// Singleton - available everywhere, tree-shakeable
@Injectable({ providedIn: 'root' })
export class GlobalService {}

// Platform-level (shared across apps)
@Injectable({ providedIn: 'platform' })
export class SharedService {}

// No providedIn - must be explicitly provided
@Injectable()
export class ScopedService {}

// Provide at component level (new instance per component)
@Component({
  providers: [ScopedService]
})
```

---

## Hierarchical Injectors

```typescript
// Parent provides service
@Component({
  selector: 'parent',
  providers: [DataService],
  template: `<child />`
})
export class ParentComponent {}

// Child gets same instance as parent
@Component({ selector: 'child' })
export class ChildComponent {
  private data = inject(DataService); // Same instance as parent
}

// Skip self - get parent's instance
export class ChildComponent {
  private data = inject(DataService, { skipSelf: true });
}

// Self only - only check this component's injector
export class ChildComponent {
  private data = inject(DataService, { self: true, optional: true });
}
```

---

## Helper Injection Functions

Create reusable inject helpers for common patterns:

```typescript
// Route param helper
export function injectParams(key: string): Signal<string | null> {

  const route = inject(ActivatedRoute);
  return toSignal(route.paramMap.pipe(map(p => p.get(key))), { initialValue: null });
}

// Usage
protected readonly $id = injectParams('id');
```

```typescript
// Query param helper
export function injectQueryParam(key: string): Signal<string | null> {

  const route = inject(ActivatedRoute);
  return toSignal(route.queryParamMap.pipe(map(p => p.get(key))), { initialValue: null });
}
```
