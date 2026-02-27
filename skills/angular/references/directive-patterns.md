# Directive Patterns

## Attribute Directive

```typescript
import { Directive, ElementRef, inject, input, effect } from '@angular/core';

@Directive({
  selector: '[highlight]',
  standalone: true
})
export class HighlightDirective {
  private el = inject(ElementRef);

  public $color = input<string>('yellow', { alias: 'highlight' });
  public $enabled = input(true, { alias: 'highlightEnabled' });

  constructor() {
    effect(() => {
      const color = this.$color();
      const enabled = this.$enabled();
      this.el.nativeElement.style.backgroundColor = enabled ? color : '';
    });
  }
}
```

Usage:

```html
<p highlight="yellow" [highlightEnabled]="true">Highlighted</p>
<p [highlight]="$color()">Dynamic</p>
```

---

## Structural Directive

```typescript
import { Directive, TemplateRef, ViewContainerRef, inject, input, effect } from '@angular/core';

@Directive({
  selector: '[ifRole]',
  standalone: true
})
export class IfRoleDirective {
  private templateRef = inject(TemplateRef);
  private viewContainer = inject(ViewContainerRef);
  private authService = inject(AuthService);

  public $role = input.required<string>({ alias: 'ifRole' });
  private hasView = false;

  constructor() {
    effect(() => {
      const required = this.$role();
      const hasRole = this.authService.$userRoles().includes(required);

      if (hasRole && !this.hasView) {
        this.viewContainer.createEmbeddedView(this.templateRef);
        this.hasView = true;
      } else if (!hasRole && this.hasView) {
        this.viewContainer.clear();
        this.hasView = false;
      }
    });
  }
}
```

Usage:

```html
<button *ifRole="'admin'">Admin Only</button>
```

---

## Host Directives

Compose directives onto a component without the consumer needing to apply them:

```typescript
@Directive({ standalone: true })
export class TooltipDirective {
  public $text = input<string>('', { alias: 'tooltip' });
  // tooltip logic...
}

@Directive({ standalone: true })
export class DisableableDirective {
  public $disabled = input(false, { alias: 'disabled' });
  // disable logic...
}

@Component({
  selector: 'fancy-button',
  standalone: true,
  hostDirectives: [
    { directive: TooltipDirective, inputs: ['tooltip'] },
    { directive: DisableableDirective, inputs: ['disabled'] }
  ],
  template: `<ng-content />`
})
export class FancyButtonComponent {}
```

Usage:

```html
<!-- tooltip and disabled are automatically available -->
<fancy-button tooltip="Click me" [disabled]="$isDisabled()">Save</fancy-button>
```

---

## Common Directive Patterns

### Auto-Focus

```typescript
@Directive({
  selector: '[autoFocus]',
  standalone: true
})
export class AutoFocusDirective {
  private el = inject(ElementRef);

  constructor() {
    afterNextRender(() => this.el.nativeElement.focus());
  }
}
```

### Click Outside

```typescript
@Directive({
  selector: '[clickOutside]',
  standalone: true
})
export class ClickOutsideDirective {
  private el = inject(ElementRef);
  public onClickOutside = output<void>();

  @HostListener('document:click', ['$event.target'])
  protected onClick(target: HTMLElement): void {

    if (!this.el.nativeElement.contains(target)) {
      this.onClickOutside.emit();
    }
  }
}
```
