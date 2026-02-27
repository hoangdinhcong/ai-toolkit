---
name: angular
description: Use when working with Angular components, signals, templates, forms, directives, or any .component.ts/.directive.ts/.html files in jms-web
---

# WorkBuddy Angular Guide

## Component Reuse Priority

1. **WorkBuddy UI** (`/jms-web/workbuddy/ui/`) - `w-button`, `w-select`, `w-checkbox`, `w-calendar`, `w-field`, `entity-dashboard`, `module-header`, ...
2. **PrimeNG v17** - `p-button`, `p-dropdown`, `p-calendar`, `p-dialog`, `p-table`, `p-menu`, ...
3. **Custom component** - last resort, follow reference patterns

## Code Conventions

- **Signals**: `$` prefix - `$count = signal(0)`, `$label = input<string>('', { alias: 'label' })`
- **Observables**: `$` suffix - `items$ = this.service.getItems()`
- **Imports**: Always `CommonModule` in imports array. Single-line imports only
- **DI**: `inject()` function, never constructor injection
- **Change detection**: Always `ChangeDetectionStrategy.OnPush`
- **Standalone**: Always `standalone: true`
- **Control flow**: `@if`/`@for`/`@switch`, never `*ngIf`/`*ngFor`
- **Styling**: LESS files with `styleUrl`. Do NOT use `:host` or inline styles. Use the component selector in LESS (e.g. `my-app { ... }`)
- **Visibility**: `public` for inputs/outputs, `protected` for template-used, `private` for internal
- **Parameters**: Single-line function parameters. Blank line after function declaration
- **Inputs/Outputs**: `input()`/`output()`/`model()` functions with `{ alias }`, never decorators
- **Bindings**: `[class.x]`/`[style.x]` direct bindings, never `ngClass`/`ngStyle`

## Quick Reference

| Topic | File | When to read |
|-------|------|-------------|
| Components | [component-patterns.md](references/component-patterns.md) | Creating/modifying components, inputs/outputs, host bindings, control flow |
| Signals | [signal-patterns.md](references/signal-patterns.md) | signal/computed/effect/linkedSignal, equality, RxJS interop |
| Data Fetching | [data-fetching-patterns.md](references/data-fetching-patterns.md) | rxResource, takeUntilDestroyed, API integration |
| Forms | [forms-patterns.md](references/forms-patterns.md) | ControlValueAccessor, reactive forms, entity pickers |
| Signal Forms | [signal-forms-patterns.md](references/signal-forms-patterns.md) | Angular 21+ experimental signal forms |
| DI | [di-patterns.md](references/di-patterns.md) | Dependency injection, InjectionToken, providers |
| Directives | [directive-patterns.md](references/directive-patterns.md) | Attribute/structural directives |
| HTTP | [http-patterns.md](references/http-patterns.md) | HttpClient, interceptors, error handling |
| Routing | [routing-patterns.md](references/routing-patterns.md) | Routes, guards, resolvers, lazy loading |
