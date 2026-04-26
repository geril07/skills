---
name: project-structure
description: Use when adding, moving, or importing files and modules; enforces role-based layer dependencies and module boundary rules
---

# Project Structure

## Overview

Code is organized by architectural role, not by framework-specific folder names. Each project should map its folders to these roles, then follow the dependency rules. Respecting these roles keeps modules decoupled and the codebase navigable.

## Roles

| Role | Purpose |
|------|---------|
| `bootstrap` | App startup, providers, routers, runtime singletons, app-wide wiring. Nothing outside bootstrap imports from it. |
| `routes` | Route, screen, controller, or endpoint composition. Glues modules and shared code into user-facing or API-facing entrypoints. |
| `modules` | Business/domain code. Each module is a cohesive domain slice. |
| `shared` | Generic primitives and technical building blocks. Move things to `modules` once they become domain-specific. |
| `globals` | Ambient constants or types injected through `globalThis`, `window`, or similar. Not imported directly. |

## Dependency Rules

Dependencies flow **downward only**:

```txt
bootstrap -> routes -> modules -> shared
```

`globals` is ambient-only. Do not import from it directly.

| From | Can import from |
|------|----------------|
| `bootstrap` | `routes`, `modules`, `shared` |
| `routes` | `modules`, `shared` |
| `modules` | `shared`, other modules' **public API only** |
| `shared` | Other files within `shared` only |
| `globals` | Ambient only; not imported |

**Violations:**
- `shared` importing from `bootstrap`, `routes`, or `modules` is **forbidden**
- `routes` importing from `bootstrap` is **forbidden**
- `modules/foo` importing internal files of `modules/bar` is **forbidden** (use `modules/bar`'s public API)

## Module Boundaries

- Import from modules only through their **public API** (usually the module's `index.ts` entrypoint)
- Do not import from internal module files across module boundaries
- Modules may depend on other modules when really needed, but only through public API

Allowed:

```ts
import { userSchema } from "@/modules/users";
```

Forbidden:

```ts
import { userSchema } from "@/modules/users/schemas/user";
```

## Shared Entry Points

Shared subfolders do not need an `index.ts`/barrel by default. Add a small scoped entrypoint only when the folder represents a cohesive primitive or package-like scope.

Good scoped entrypoint:

```ts
import { Table } from "@/shared/table";
```

Good direct import:

```ts
import { formatDate } from "@/shared/date/formatDate";
```

Avoid broad catch-all barrels:

```ts
import { formatDate, Table, cn } from "@/shared";
```

## Grouping

Prefer flat, scope-based grouping. Keep related files together under the smallest meaningful scope, and add subfolders only when the scope becomes too large to scan.

Prefer:

```txt
shared/table/
  Table.tsx
  columns.ts
  schema.ts
  constants.ts
```

Avoid splitting one concern across type buckets:

```txt
shared/components/Table.tsx
shared/schemas/table.ts
shared/constants/table.ts
```

Type-based files are fine inside an already clear scope when they describe that scope:

```txt
modules/users/
  index.ts
  schema.ts
  types.ts
```

## Example Structure

```txt
src/
  app/
    main.tsx
    router/
    providers/
    styles/
  pages/
    roles/
      page.tsx
    todos/
      page.tsx
  modules/
    auth/
      index.ts
    session/
      index.ts
    users/
      index.ts
    roles/
      index.ts
    todos/
      index.ts
  shared/
    api/
    assets/
    ui/
    table/
      index.ts
      Table.tsx
    date/
      formatDate.ts
```

In this example, `src/app/` is the bootstrap role, `src/pages/` is the routes role, `src/modules/` is the modules role, and `src/shared/` is the shared role. Other projects may use different folder names for the same roles.

## Red Flags

- Importing from bootstrap code outside of bootstrap
- Importing an internal path from another module (e.g., `modules/users/internal/helpers`)
- Putting domain logic in `shared`
- `shared` file importing from bootstrap, routes, or modules
- Adding a file without identifying its architectural role
