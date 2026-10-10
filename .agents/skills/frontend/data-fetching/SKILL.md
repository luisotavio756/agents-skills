---
name: data-fetching
description: >-
  React Query patterns for data fetching. Use when creating queries, mutations,
  cache invalidation, optimistic updates, pagination, or when the user mentions
  useQuery, useMutation, React Query, TanStack Query, or data fetching hooks.
---

# Data Fetching with React Query

Adapt to the project's React Query version (v4 or v5 — v5 uses a single object arg). When project-specific patterns exist, follow those first.

## Principles

1. **Server state stays out of Zustand** — React Query owns fetched data; Zustand owns client/UI state.
2. **Wrap every query in a custom hook** — components never call `useQuery` directly.
3. **Co-locate query keys** — define them in a `query-keys.ts` file per domain, not inline.
4. **Type the response** — always type `queryFn` return; use Zod to parse at the boundary.

## Query key factory

Define keys in `{domain}/query-keys.ts`:

```typescript
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};
```

## Custom query hook

Wrap `useQuery` in a domain hook — never call it directly from a component:

```typescript
export function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => UserService.getById(id),
    enabled: !!id,
  });
}
```

## Custom mutation hook

```typescript
export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (payload: UpdateUserPayload) => UserService.update(payload),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(id) });
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}
```

## Optimistic updates

Use when the operation is likely to succeed and you want instant feedback:

```typescript
export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => ItemService.toggleFavorite(id),
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: itemKeys.detail(id) });

      const previous = queryClient.getQueryData<Item>(itemKeys.detail(id));

      queryClient.setQueryData<Item>(itemKeys.detail(id), (old) =>
        old ? { ...old, isFavorite: !old.isFavorite } : old
      );

      return { previous };
    },
    onError: (_, id, ctx) => {
      if (ctx?.previous) {
        queryClient.setQueryData(itemKeys.detail(id), ctx.previous);
      }
    },
    onSettled: (_, __, id) => {
      queryClient.invalidateQueries({ queryKey: itemKeys.detail(id) });
    },
  });
}
```

## Pagination

```typescript
export function useUsers(filters: UserFilters) {
  return useInfiniteQuery({
    queryKey: userKeys.list(filters),
    queryFn: ({ pageParam = 1 }) =>
      UserService.list({ ...filters, page: pageParam }),
    getNextPageParam: (last) =>
      last.hasNextPage ? last.page + 1 : undefined,
    initialPageParam: 1,
  });
}

// In the component:
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useUsers(filters);
const items = data?.pages.flatMap((p) => p.items) ?? [];
```

## Error handling

- Set a global `onError` in `QueryClient` for generic API errors (401, 500)
- Override per-query for domain-specific errors

```typescript
// query-client.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) =>
        isNetworkError(error) && failureCount < 2,
      staleTime: 1000 * 60, // 1 min default
    },
    mutations: {
      onError: (error) => {
        toast.error(toUserMessage(error));
      },
    },
  },
});
```

## Render states in components

Destructure from the hook, not from `useQuery` directly:

```typescript
function UserProfile({ id }: { id: string }) {
  const { data: user, isLoading, error } = useUser(id);

  if (isLoading) return <UserProfileSkeleton />;
  if (error) return <ErrorMessage message={error.message} />;
  if (!user) return null;

  return <UserCard user={user} />;
}
```

## Prefetching

Prefetch on hover or route entry to eliminate loading states:

```typescript
function prefetchUser(id: string) {
  return queryClient.prefetchQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => UserService.getById(id),
  });
}
```

## Cache invalidation patterns

| Pattern | When to use |
|---------|------------|
| `invalidateQueries` | Most mutations — refetch from server |
| `setQueryData` | Optimistic update or response already contains new data |
| `removeQueries` | User signs out or entity is deleted |
| `resetQueries` | Reset to initial state (e.g. after a filter clear) |

## Avoid

- Calling `useQuery` directly in components — wrap in a custom hook
- Inline query key strings — use the key factory
- Storing API responses in Zustand when React Query cache is sufficient
- `refetchInterval` polling when WebSocket or SSE is available
- Disabling `staleTime` globally (causes unnecessary refetches on focus)

## Checklist

```
- [ ] Query keys defined in query-keys.ts using a factory
- [ ] useQuery / useMutation wrapped in a custom domain hook
- [ ] queryFn return type is explicit (or inferred from Zod parse)
- [ ] Loading, error, and empty states handled in component
- [ ] Mutations invalidate or update relevant query cache on success
- [ ] Global error handler set in QueryClient config
```
