# Structure guide

## Top-level layers:

- app/ - app bootstrap, providers, router, runtime singletons, app-wide wiring; no one outside app/ should import from it
- pages/ - route/screen composition layer
- modules/ - business/domain-related code
- shared/ - generic primitives and technical building blocks; move things to modules/ once they become domain-specific
- globals/ - optional; global constants or dts injected through globalThis, window, or similar

## Dependencies:

- app/ can use pages/, modules/, and shared/
- pages/ can use modules/ and shared/
- modules/ can use shared/
- modules/ may use other modules only through their public API when really needed
- shared/ must not depend on modules/ or app/
- globals/ are ambient and are not imported directly

### Module boundaries:

- import from modules only through their public API
- do not import from internal module files across module boundaries

## Recommendations:

- Prefer structuring by scope over structuring by type when several files belong to the same concern:
  shared/schemas/table.ts + shared/types/table.ts -> shared/table/{schemas/types/constants}.ts
  Type-based grouping inside an already clear scope (e.g. modules/users/{types/schemas}) is also fine

## Example:

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
