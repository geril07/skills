# Layered Modular Layout Reference

## Roles

| Role | Purpose |
|------|---------|
| `bootstrap` | Project startup and runtime wiring: providers, routers, dependency setup, process or app entrypoints. Nothing outside bootstrap imports from it. |
| `routes` | User-facing or API-facing composition: routes, screens, pages, route handlers, endpoint adapters, or framework bindings. |
| `modules` | Business/domain slices. Each module owns a cohesive area of behavior, rules, data access, and domain-specific UI. |
| `shared` | Generic primitives and technical building blocks reusable across domains. Move code to `modules` once it becomes domain-specific. |
| `globals` | Ambient declarations or values exposed through `globalThis`, `window`, or similar runtime globals. Not imported directly. |

## Dependency Rules

Dependencies flow **downward only**:

```txt
bootstrap -> routes -> modules -> shared
```

`globals` is ambient-only. Do not import from it directly.

Folder names are project-specific. Apply these rules by the role a file plays, not by the literal folder name.

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

One possible source layout:

```txt
src/
  bootstrap/
    main.ts
    router.ts
    providers.ts
  routes/
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

## Red Flags

- Importing from bootstrap code outside of bootstrap
- Importing an internal path from another module (e.g., `modules/users/internal/helpers`)
- Putting domain logic in `shared`
- `shared` file importing from bootstrap, routes, or modules
- Putting business rules directly in routes instead of composing modules
- Adding a file without identifying its architectural role
