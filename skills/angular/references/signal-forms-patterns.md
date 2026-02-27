# Signal Forms Patterns

> **EXPERIMENTAL / NOT YET ADOPTED** - Angular 21+ signal-based forms. Not yet used in WorkBuddy production code. Included for reference when the team adopts this API.

## Basic Usage

```typescript
import { signal } from '@angular/core';
import { form, FormField, required, email, minLength, validate, customError, submit } from '@angular/forms/signals';

export class LoginComponent {
  protected $model = signal({ email: '', password: '' });

  protected loginForm = form(this.$model, (s) => {
    required(s.email);
    email(s.email);
    required(s.password);
    minLength(s.password, 8);
  });

  protected async onSubmit(): Promise<void> {

    await submit(this.loginForm, async (f) => {
      await this.authService.login(f().value());
      return null;
    });
  }
}
```

### Template

```html
<form (submit)="onSubmit()">
  <input type="email" [formField]="loginForm.email" />
  <input type="password" [formField]="loginForm.password" />
  <button [disabled]="loginForm().invalid()">Login</button>
</form>
```

---

## Validators

| Validator | Usage |
|-----------|-------|
| `required(field)` | Field must have value |
| `email(field)` | Valid email format |
| `min(field, n)` / `max(field, n)` | Numeric bounds |
| `minLength(field, n)` / `maxLength(field, n)` | String length |
| `pattern(field, regex)` | Regex validation |
| `validate(field, fn)` | Custom sync validator |
| `validateHttp(field, config)` | Async HTTP validation |

---

## Cross-Field Validation

```typescript
const myForm = form(model, (s) => {
  validate(s.confirmPassword, ({ value, valueOf }) =>
    value() !== valueOf(s.password) ? customError({ kind: 'mismatch', message: 'Passwords must match' }) : null
  );
});
```

---

## Reusable Schemas

```typescript
import { schema, apply, applyEach } from '@angular/forms/signals';

const addressSchema = schema<Address>((a) => {
  required(a.street);
  required(a.city);
  required(a.zip);
});

const itemSchema = schema<OrderItem>((i) => {
  required(i.name);
  min(i.quantity, 1);
});

const orderForm = form(model, (s) => {
  apply(s.shipping, addressSchema);
  applyEach(s.items, itemSchema);
});
```

---

## Field State API

```typescript
myForm.email().value();      // Current value
myForm.email().value.set();  // Set value
myForm.email().valid();      // Is valid
myForm.email().invalid();    // Is invalid
myForm.email().errors();     // Error array [{kind, message}]
myForm.email().touched();    // User interacted
myForm.email().dirty();      // Value changed
myForm.email().pending();    // Async validation in progress
```

---

## FormValueControl (Custom Controls)

Replaces `ControlValueAccessor` for signal forms.

```typescript
import { model, input } from '@angular/core';
import { FormValueControl, ValidationError } from '@angular/forms/signals';

@Component({
  selector: 'star-rating',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @for (i of [1,2,3,4,5]; track i) {
      <button type="button" (click)="value.set(i)" [disabled]="disabled()">
        {{ i <= value() ? '★' : '☆' }}
      </button>
    }
  `
})
export class StarRatingComponent implements FormValueControl<number> {
  readonly value = model(0);
  readonly disabled = input(false);
  readonly errors = input<readonly ValidationError[]>([]);
}
```

Usage:

```html
<star-rating [field]="myForm.rating" />
```
