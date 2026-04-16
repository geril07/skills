---
name: project-structure
description: Use when adding, moving, or importing files and modules — enforces layer dependencies and module boundary rules
---

# Project Structure

## Overview

Code is organized into layers with strict dependency rules. Respecting these layers keeps modules decoupled and the codebase navigable.

## Layers

| Layer | Purpose |
|-------|---------|
| `app/` | Bootstrap, providers, router, runtime singletons, app-wide wiring. Nothing outside `app/` imports from it. |
| `pages/` | Route/screen composition layer. Glues modules and shared code into viewable screens. |
| `modules/` | Business/domain code. Each module is a cohesive domain slice. |
| `shared/` | Generic primitives and technical building blocks. Move things to `modules/` once they become domain-specific. |
| `globals/` | Ambient constants or types injected through `globalThis`, `window`, or similar. Not imported directly. |

## Dependency Rules

Dependencies flow **downward only**:

```
app/ → pages/ → modules/ → shared/
                          ↓
                       globals/ (ambient, not imported)
```

| From | Can import from |
|------|----------------|
| `app/` | `pages/`, `modules/`, `shared/` |
| `pages/` | `modules/`, `shared/` |
| `modules/` | `shared/`, other modules' **public API only** |
| `shared/` | Nothing outside `shared/` |
| `globals/` | Not imported — ambient only |

**Violations:**
- `shared/` importing from `modules/` or `app/` → **forbidden**
- `pages/` importing from `app/` → **forbidden**
- `modules/foo` importing internal files of `modules/bar` → **forbidden** (use `modules/bar`'s public API)

## Module Boundaries

- Import from modules only through their **public API** (the module's index/barrel file)
- Do not import from internal module files across module boundaries
- Modules may depend on other modules when really needed, but only through public API

## Grouping Recommendation

Prefer **scope-based grouping** over type-based grouping when files belong to the same concern:

```
shared/schemas/table.ts + shared/types/table.ts
→ shared/table/{schemas,types,constants}.ts
```

Type-based grouping inside an already clear scope is fine:
```
modules/users/{types,schemas}  ✓
```

## Example Structure

```
src/
  app/
    main.tsx
    router/
    query/
    styles/
    providers/
  pages/
    Roles/
    Todos/
  modules/
    auth/
    session/
    users/
    roles/
    todos/
  shared/
    api/
    assets/
    components/
    hooks/
    utils/
    types/
```

## Red Flags — STOP and Reconsider

- Importing from `app/` outside of `app/`
- Importing an internal path from another module (e.g., `modules/users/internal/helpers`)
- Putting domain logic in `shared/`
- `shared/` file importing from `modules/`
- Adding a file that doesn't fit any layer's purpose