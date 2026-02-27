# Component Advanced Patterns

## viewChild / contentChild Signal Queries

```typescript
import { viewChild, viewChildren, contentChild, contentChildren, ElementRef, TemplateRef } from '@angular/core';

export class MyComponent {
  // Single element query
  protected $input = viewChild<ElementRef>('inputRef');
  protected $inputRequired = viewChild.required<ElementRef>('inputRef');

  // Multiple elements
  protected $items = viewChildren<ElementRef>('itemRef');

  // Component query
  protected $dialog = viewChild(DialogComponent);

  // Content projection queries
  protected $header = contentChild<TemplateRef<unknown>>('header');
  protected $tabs = contentChildren(TabComponent);

  constructor() {
    effect(() => {
      const input = this.$input();
      if (input) input.nativeElement.focus();
    });
  }
}
```

### Template

```html
<input #inputRef />
<div #itemRef *ngFor="let item of items">{{ item }}</div>
```

---

## Content Projection

### Single Slot

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content />
    </div>
  `
})
```

### Named Slots

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-header"><ng-content select="[card-header]" /></div>
      <div class="card-body"><ng-content /></div>
      <div class="card-footer"><ng-content select="[card-footer]" /></div>
    </div>
  `
})
```

Usage:

```html
<app-card>
  <h2 card-header>Title</h2>
  <p>Body content</p>
  <button card-footer>Action</button>
</app-card>
```

### Conditional Projection

```typescript
export class TabsComponent {
  protected $tabs = contentChildren(TabComponent);
  protected $activeIndex = signal(0);
  protected $activeTab = computed(() => this.$tabs()[this.$activeIndex()] ?? null);
}
```

---

## @defer - Lazy Loading Blocks

```html
<!-- Load when visible -->
@defer (on viewport) {
  <heavy-chart [data]="$data()" />
} @placeholder {
  <div class="chart-placeholder">Chart loading area</div>
} @loading (minimum 300ms) {
  <loading-spinner />
} @error {
  <p>Failed to load chart</p>
}

<!-- Load on interaction -->
@defer (on interaction) {
  <comments-section />
} @placeholder {
  <button>Load Comments</button>
}

<!-- Load on condition -->
@defer (when $showDetails()) {
  <detail-panel />
}

<!-- Prefetch -->
@defer (on viewport; prefetch on idle) {
  <recommendation-engine />
}
```

### Triggers

| Trigger | When |
|---------|------|
| `on viewport` | Element enters viewport |
| `on interaction` | User clicks/focuses placeholder |
| `on idle` | Browser is idle |
| `on timer(500ms)` | After delay |
| `on hover` | Mouse enters placeholder |
| `when condition` | Expression becomes truthy |

---

## Dynamic Components

```typescript
import { ViewContainerRef, Type } from '@angular/core';

export class DynamicHostComponent {
  protected $container = viewChild.required('container', { read: ViewContainerRef });

  protected loadComponent(component: Type<unknown>, inputs?: Record<string, unknown>): void {

    const container = this.$container();
    container.clear();
    const ref = container.createComponent(component);
    if (inputs) {
      Object.entries(inputs).forEach(([key, value]) => ref.setInput(key, value));
    }
  }
}
```

Template:

```html
<ng-container #container />
```

---

## Component Communication Patterns

### Parent to Child - Inputs

```typescript
// Parent template
<child-comp [data]="$parentData()" />

// Child
public $data = input.required<Data>({ alias: 'data' });
```

### Child to Parent - Outputs

```typescript
// Child
public onSave = output<Data>();
this.onSave.emit(data);

// Parent template
<child-comp (onSave)="handleSave($event)" />
```

### Two-Way Binding - Model

```typescript
// Child
public $value = model<string>('', { alias: 'value' });

// Parent template
<child-comp [(value)]="$parentValue" />
```

### Service-Based (Siblings/Unrelated)

```typescript
@Injectable({ providedIn: 'root' })
export class SharedStateService {
  private readonly _$selection = signal<string | null>(null);
  public readonly $selection = this._$selection.asReadonly();

  public select(id: string): void {

    this._$selection.set(id);
  }
}
```
