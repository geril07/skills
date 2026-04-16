# Naming Guide

## Rules

- Prefer local filenames unless context improves clarity
- Prefer scoped exported names when code crosses file or module boundaries

## Examples

- `users/schema.ts` -> `userSchema`
- `reports/query.ts` -> `buildReportQuery`
- `orders/types.ts` -> `OrderFilters`

- `pages/Home/Dashboard.tsx` -> `Dashboard`
- `pages/Home/Dashboard.tsx` -> `HomeDashboard` when broader usage needs more context

- `shared/formatting/date.ts` -> `formatDate`
- `shared/storage/session.ts` -> `readSession`
- `shared/validation/email.ts` -> `emailSchema`
- `shared/table/constants.ts` -> `pageSizeOptions`
