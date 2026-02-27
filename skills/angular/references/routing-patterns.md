# Routing Patterns

## Route Configuration

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
  { path: 'dashboard', component: DashboardComponent },
  { path: 'items/:id', component: ItemDetailComponent },
  { path: '**', component: NotFoundComponent }
];
```

---

## Lazy Loading

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)
  },
  {
    path: 'settings',
    loadChildren: () => import('./settings/settings.routes').then(m => m.SETTINGS_ROUTES)
  }
];
```

---

## Functional Guards

```typescript
import { CanActivateFn, CanDeactivateFn, Router } from '@angular/router';
import { inject } from '@angular/core';

// Auth guard
export const authGuard: CanActivateFn = (route, state) => {

  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) return true;
  return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};

// Role guard
export const roleGuard: CanActivateFn = (route) => {

  const authService = inject(AuthService);
  const requiredRole = route.data['role'] as string;
  return authService.$userRoles().includes(requiredRole);
};

// Unsaved changes guard
export const unsavedChangesGuard: CanDeactivateFn<{ hasUnsavedChanges: () => boolean }> = (component) => {

  if (component.hasUnsavedChanges()) {
    return confirm('You have unsaved changes. Leave?');
  }
  return true;
};
```

### Apply Guards

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard],
    data: { role: 'admin' },
    component: AdminComponent
  },
  {
    path: 'editor',
    canDeactivate: [unsavedChangesGuard],
    component: EditorComponent
  }
];
```

---

## Resolvers

```typescript
import { ResolveFn } from '@angular/router';

export const itemResolver: ResolveFn<Item | null> = (route) => {

  const itemService = inject(ItemService);
  const id = route.paramMap.get('id');
  return id ? itemService.getById(id).pipe(catchError(() => of(null))) : of(null);
};
```

### Apply Resolver

```typescript
{
  path: 'items/:id',
  component: ItemDetailComponent,
  resolve: { item: itemResolver }
}

// Access in component
export class ItemDetailComponent {
  private route = inject(ActivatedRoute);
  protected $item = toSignal(this.route.data.pipe(map(d => d['item'] as Item)), { initialValue: null });
}
```

---

## Route Params as Signals

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

export class DetailComponent {
  private route = inject(ActivatedRoute);

  // Route param
  protected $id = toSignal(this.route.paramMap.pipe(map(p => p.get('id'))), { initialValue: null });

  // Query param
  protected $tab = toSignal(this.route.queryParamMap.pipe(map(p => p.get('tab'))), { initialValue: 'overview' });

  // Or use helper (see di-patterns.md)
  protected $id2 = injectParams('id');
}
```

---

## Navigation

```typescript
export class NavComponent {
  private router = inject(Router);

  protected goToItem(id: string): void {

    this.router.navigate(['/items', id]);
  }

  protected goWithQuery(): void {

    this.router.navigate(['/search'], { queryParams: { q: 'test', page: 1 } });
  }

  protected goRelative(): void {

    this.router.navigate(['../sibling'], { relativeTo: this.route });
  }
}
```

### Template Navigation

```html
<a [routerLink]="['/items', $id()]">View Item</a>
<a routerLink="/dashboard" routerLinkActive="active">Dashboard</a>
```
