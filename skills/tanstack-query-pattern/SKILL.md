---
name: tanstack-query-pattern
description: Use when writing or refactoring TanStack Query queries, mutations, query keys, disabled queries, and invalidation logic
---

# TanStack Query Pattern

## Overview

Use `queryOptions` and `mutationOptions` factories as the source of truth for TanStack Query configuration. Prefer the smallest correct query key shape, and add disabled behavior, key normalization, or broad invalidation scopes only when the query actually needs them.

## Query Factories

Define reusable `queryOptions` factories next to the domain API they represent.

Default shape for a query with params:

```ts
import { queryOptions } from "@tanstack/react-query";

type ListTodosArg = {
  userId: number;
};

export const listTodosQueryOptions = (arg: ListTodosArg) =>
  queryOptions({
    queryKey: ["listTodos", arg] as const,
    queryFn: () => fetchTodos(arg),
  });
```

Default shape for a query without params:

```ts
export const currentUserQueryOptions = () =>
  queryOptions({
    queryKey: ["currentUser"] as const,
    queryFn: fetchCurrentUser,
  });
```

Rules:

1. Use one object argument when a query has params.
2. Type the arg inline for small shapes; extract a type when reused or large.
3. Do not include implementation names like `"QueryOptions"` in query keys.
4. Reuse `queryOptions(...).queryKey` for exact query operations such as `setQueryData`, `getQueryData`, `prefetchQuery`, and exact invalidation.

## Disabled Queries

Use `skipToken` only when a query is called before all required params are available.

```ts
import { queryOptions, skipToken } from "@tanstack/react-query";

export const listTodosQueryOptions = (arg: ListTodosArg | undefined) =>
  queryOptions({
    queryKey: ["listTodos", arg] as const,
    queryFn: arg ? () => fetchTodos(arg) : skipToken,
  });
```

Notes:

1. `skipToken` creates a disabled query state.
2. Disabled queries do not fetch when mounted or invalidated.
3. `refetch()` does not work meaningfully with `skipToken`; it results in a missing query function error state.
4. Do not treat a disabled/no-arg query key as meaningful cache identity.

## Key Normalization

Use `{ ...arg }` only when missing args and empty filters should intentionally become the same key shape.

Default key preserves the arg value:

```ts
queryKey: ["listTodos", arg] as const;
```

Normalized only when intended:

```ts
queryKey: ["listTodos", { ...arg }] as const;
```

The normalized form intentionally collapses these inputs:

```ts
undefined -> {}
{} -> {}
```

TanStack Query hashes query keys by value, so recreating object params does not make a key unstable. Object key order is not a reason to spread args.

## Broad Invalidation Scopes

Add hierarchical key scopes only when broad matching is needed.

Use this when you need to invalidate all todo lists without invalidating todo details:

```ts
const todosKeys = {
  lists: () => ["todos", "list"] as const,
  list: (arg: ListTodosArg) => [...todosKeys.lists(), arg] as const,
  detail: (id: number) => ["todos", "detail", id] as const,
};

export const listTodosQueryOptions = (arg: ListTodosArg) =>
  queryOptions({
    queryKey: todosKeys.list(arg),
    queryFn: () => fetchTodos(arg),
  });
```

Then exact invalidation uses the query factory:

```ts
client.invalidateQueries({
  queryKey: listTodosQueryOptions({ userId }).queryKey,
});
```

Broad invalidation uses the explicit scope utility:

```ts
client.invalidateQueries({
  queryKey: todosKeys.lists(),
});
```

## Mutation Factories

Prefer `mutationOptions` factories as the mutation source of truth.

```ts
import { mutationOptions } from "@tanstack/react-query";

type AddTodoVariables = {
  name: string;
  userId: number;
};

export const addTodoMutationOptions = () =>
  mutationOptions({
    mutationFn: ({ name, userId }: AddTodoVariables) => addTodo({ name, userId }),
    onSuccess: (_data, variables, _onMutateResult, { client }) => {
      return client.invalidateQueries({
        queryKey: listTodosQueryOptions({ userId: variables.userId }).queryKey,
      });
    },
  });
```

Rules:

1. Return or await invalidation promises from mutation callbacks when mutation completion should wait for them.
2. Use `Promise.all` for multiple invalidations that can run in parallel.
3. Use query option factories for exact invalidation keys instead of hard-coded duplicate keys.
4. If invalidation logic grows, extract a small colocated invalidation utility near the query factories.
5. If a mutation needs hook-level dependencies, either accept them in the mutation factory or define a small hook wrapper.

## Decision Order

1. Start with a `queryOptions` or `mutationOptions` factory.
2. Use a single object arg when params exist.
3. Use `queryKey: ["name", arg]` as the default key shape.
4. Add `skipToken` only when disabled behavior is required.
5. Add `{ ...arg }` only when empty/missing args should normalize to the same object key shape.
6. Add scoped keys only when broad invalidation or cache operations need prefix matching.
7. Extract invalidation utilities only after repeated invalidation logic appears.

## Red Flags

- Hard-coded query keys duplicated across queries and mutations
- Passing `undefined` into a query that should not be disabled
- Large mutation `onSuccess` blocks with repeated invalidation details
