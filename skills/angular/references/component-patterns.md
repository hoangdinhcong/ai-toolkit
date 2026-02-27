# Component Patterns

## Component Structure

```typescript
import { Component, ChangeDetectionStrategy, ViewEncapsulation, signal, computed, effect, inject, input, output, model } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-example',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  encapsulation: ViewEncapsulation.None,
  imports: [CommonModule],
  templateUrl: './example.component.html',
  styleUrl: './example.component.less',
  host: {
    '[class]': '$hostClass()',
    '[class.disabled]': '$disabled()',
    '(click)': 'handleClick($event)'
  }
})
export class ExampleComponent {
  // DI
  protected readonly service = inject(MyService);

  // Inputs (public - external API)
  public $label = input<string>('', { alias: 'label' });
  public $size = input<'small' | 'medium' | 'large'>('medium', { alias: 'size' });
  public $disabled = input(false, { alias: 'disabled' });

  // Outputs
  public onValueChanged = output<string>();

  // Model (two-way binding)
  public $checked = model<boolean>(false, { alias: 'checked' });

  // Local state (protected - template only)
  protected $count = signal(0);

  // Computed
  protected $display = computed(() => `${this.$label()} (${this.$count()})`);

  // Host class
  protected $hostClass = computed(() => {
    const classes = { 'is-small': this.$size() === 'small', 'is-disabled': this.$disabled() };
    return Object.keys(classes).filter(k => classes[k]).join(' ');
  });

  // Effects in constructor
  constructor() {
    effect(() => {
      const label = this.$label();
      console.log('Label changed:', label);
    });
  }

  // Methods (protected if template-only)
  protected handleClick(event: Event): void {

    this.$count.update(c => c + 1);
    this.onValueChanged.emit(this.$display());
  }
}
```

## Signal Inputs

```typescript
// Required input
public $userId = input.required<string>({ alias: 'userId' });

// Optional with default
public $label = input<string>('', { alias: 'label' });

// Boolean shorthand
public $disabled = input(false, { alias: 'disabled' });

// Transform
public $count = input(0, { alias: 'count', transform: numberAttribute });
```

## Signal Outputs

```typescript
// Basic output
public onSave = output<FormData>();

// Emit
this.onSave.emit(data);
```

## Model (Two-Way Binding)

```typescript
// In component
public $value = model<string>('', { alias: 'value' });

// In template usage: <my-comp [(value)]="parentSignal" />
```

## Host Bindings

Use the `host` object in `@Component`, never `@HostBinding`/`@HostListener`:

```typescript
@Component({
  host: {
    '[class]': '$hostClass()',           // Dynamic class list
    '[class.active]': '$isActive()',     // Boolean class toggle
    '[style.width.px]': '$width()',      // Style binding
    '[attr.aria-label]': '$ariaLabel()', // Attribute binding
    '(click)': 'onClick($event)',        // Event listener
    '(keydown.enter)': 'onEnter($event)'
  }
})
```

## Native Control Flow

### @if

```html
@if ($user(); as user) {
  <span>{{ user.name }}</span>
} @else if ($isLoading()) {
  <loading-spinner />
} @else {
  <empty-state />
}
```

### @for

```html
@for (item of $items(); track item._id) {
  <div>{{ $index + 1 }}. {{ item.name }}</div>
  @if ($first) { <span class="badge">First</span> }
} @empty {
  <p>No items found</p>
}
```

### @switch

```html
@switch ($viewMode()) {
  @case ('grid') { <grid-view /> }
  @case ('list') { <list-view /> }
  @default { <default-view /> }
}
```

### @let (Local Variables)

```html
@let viewService = $viewService();
@let userName = $user().profile.name;

<h1>{{ userName }}</h1>
<div>{{ viewService.getData() }}</div>
```

## Styling

- Always use LESS files (`.less`)
- Use `styleUrl` (singular), not `styles`
- Do NOT use `:host` - use the component selector instead
- Use `ViewEncapsulation.None` when styles need to affect child components

```less
// BAD - don't use :host
:host { display: block; }

// GOOD - use component selector
app-example {
  display: block;

  .inner {
    padding: 8px 12px;
    background: var(--surface-100);

    &:hover {
      background: var(--surface-200);
    }
  }

  &-large {
    padding: 16px 24px;
  }
}
```

## Image Optimization

Use `NgOptimizedImage` for images:

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [CommonModule, NgOptimizedImage],
  template: `<img ngSrc="/assets/logo.png" width="200" height="100" priority />`
})
```

## Accessibility

- Use semantic HTML (`<button>`, `<nav>`, `<main>`)
- Add `aria-label` via host bindings for interactive components
- Support keyboard navigation (`keydown.enter`, `keydown.space`)

```typescript
host: {
  'role': 'button',
  '[attr.aria-disabled]': '$disabled()',
  '[attr.aria-label]': '$ariaLabel()',
  '[tabindex]': '$disabled() ? -1 : 0',
  '(keydown.enter)': 'handleAction($event)',
  '(keydown.space)': 'handleAction($event)'
}
```

---

For advanced patterns (viewChild, contentChild, @defer, dynamic components, content projection), see [component-patterns-advanced.md](component-patterns-advanced.md)
