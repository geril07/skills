---
name: layered-modular-layout
description: Organize source code by codebase role and enforce dependency boundaries. Use when adding, moving, or importing files and modules.
---

# Layered Modular Layout

## Quick start

Organize source code by role, not by framework-specific folder names. Dependencies flow downward only:

```txt
src/
  bootstrap/
  routes/
  modules/
  shared/
```

```ts
import { userSchema } from "@/modules/users"; // public API only
```

Before creating, moving, or importing a file, identify its role and keep dependencies flowing from outer entrypoints toward inner reusable code.

## Workflows

1. **Identify the file's role** (`bootstrap`, `routes`, `modules`, `shared`, or `globals`)
2. **Place the file** under the matching layer and smallest meaningful scope
3. **Import only downward** — `bootstrap` may import from `routes`, `modules`, `shared`; `routes` from `modules`, `shared`; `modules` from `shared` and other modules' public APIs; `shared` only from `shared`
4. **Use public APIs** across module boundaries (e.g., `modules/users/index.ts`), never internal paths
5. **Group by scope**, not by type; add subfolders only when a scope becomes too large to scan
6. **Avoid red flags**: never import from `bootstrap` outside bootstrap, never import internal module paths, never put domain logic in `shared`

## Advanced features

See [REFERENCE.md](REFERENCE.md) for the full role matrix, dependency rules, module boundaries, grouping guidance, example structure, and red flags.
