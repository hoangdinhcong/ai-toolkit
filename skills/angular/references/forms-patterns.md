# Forms Patterns

## ControlValueAccessor with isWrite Pattern

The `isWrite` flag prevents the effect from emitting back to the form when the form writes a value.

```typescript
import { Component, ChangeDetectionStrategy, effect, input, model } from '@angular/core';
import { NG_VALUE_ACCESSOR, ControlValueAccessor } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'custom-input',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule],
  templateUrl: './custom-input.component.html',
  styleUrl: './custom-input.component.less',
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: CustomInputComponent,
    multi: true
  }]
})
export class CustomInputComponent implements ControlValueAccessor {
  public $label = input<string>('', { alias: 'label' });
  public $placeholder = input<string>('', { alias: 'placeholder' });
  public $value = model<string>('', { alias: 'value' });
  public $disabled = model(false, { alias: 'disabled' });

  private changeFn: (value: any) => void;
  private touchedFn: () => void;
  private isWrite = false;

  writeValue(value: string): void {

    this.isWrite = true;
    this.$value.set(value ?? '');
  }

  registerOnChange(fn: any): void { this.changeFn = fn; }
  registerOnTouched(fn: any): void { this.touchedFn = fn; }
  setDisabledState(disabled: boolean): void { this.$disabled.set(disabled); }

  constructor() {
    effect(() => {
      const value = this.$value();
      if (this.isWrite) { this.isWrite = false; return; }
      if (this.changeFn) this.changeFn(value);
    });
  }

  protected handleInput(event: Event): void {

    this.$value.set((event.target as HTMLInputElement).value);
  }

  protected handleBlur(): void {

    if (this.touchedFn) this.touchedFn();
  }
}
```

---

## Entity Picker Usage

```html
<w-select
  formControlName="contactId"
  entityPicker="contact"
  [entityParentId]="$accountId()"
  [entityOptions]="{
    addText: 'Select Contact',
    loadType: 'lazy'
  }"
  placeholder="Choose a contact"
/>

<!-- Multiple selection -->
<w-select
  formControlName="assigneeIds"
  entityPicker="user"
  [entityOptions]="{ addText: 'Select Assignees', multiple: true }"
  placeholder="Choose assignees"
/>
```

---

## Reactive Forms Basics

```typescript
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';

export class MyFormComponent {
  private fb = inject(FormBuilder);

  protected form: FormGroup = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    age: [null, [Validators.min(0), Validators.max(150)]],
    address: this.fb.group({
      street: [''],
      city: ['', Validators.required]
    })
  });

  protected async onSubmit(): Promise<void> {

    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }
    const data = this.form.getRawValue();
    // save...
  }
}
```

### Template Error Display

```html
@if (form.get('email').touched && form.get('email').invalid) {
  @if (form.get('email').hasError('required')) {
    <span class="error">Email is required</span>
  } @else if (form.get('email').hasError('email')) {
    <span class="error">Invalid email format</span>
  }
}
```

---

## Decision Guide

| Scenario | Pattern |
|----------|---------|
| Custom form control for reactive forms | `ControlValueAccessor` with `isWrite` pattern |
| Entity selection (contacts, jobs, users) | `entityPicker` directive on `w-select` |
| Standard forms with validation | Reactive Forms with `FormBuilder` |
| Angular 21+ new projects | Signal Forms (see [signal-forms-patterns.md](signal-forms-patterns.md)) |
