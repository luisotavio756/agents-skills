---
name: zustand-store
description: >-
  Organize Zustand stores with separated actions, selectors, typed hooks, and
  optional staging stores. Use when creating or modifying Zustand stores, store
  actions, selectors, edit/cancel flows, or when the user mentions zustand,
  store structure, or useXxxActions.
---

# Zustand Store Organization

Adapt to the project's store location and naming (e.g. `src/store/`, `src/stores/`, `src/state/`). When project-specific rules exist (e.g. `zustand-store.mdc`), follow those first — this skill covers the structural conventions.

## Principles

1. **One folder per store domain** — separate concerns into independent stores rather than one global store.
2. **Actions live outside the store file** — the store file defines state shape and wires hooks; mutation logic sits in `actions/`.
3. **Components never call `set` directly** — expose `useXxxActions()` (and granular sub-hooks when needed).
4. **Selectors for reads** — prefer dedicated `use*` hooks for subscriptions and `get*` helpers for callbacks/validators.
5. **Server state stays out of Zustand** — keep fetched/cached API data in your data-fetching layer (React Query, SWR, etc.); use Zustand for client/UI/editing state.

## Directory layout

```
{storeRoot}/{domain}/
├── {domain}.store.ts          # state type, create(), hook exports
├── actions/
│   ├── {domain}.actions.ts    # action factory + action types
│   ├── index.ts
│   ├── {domain}.actions.test.ts
│   └── {feature}/             # optional, for large/composite stores
│       ├── v1.actions.ts
│       └── index.ts
├── selectors/
│   ├── {domain}.selectors.ts
│   └── index.ts
└── index.ts                   # optional barrel
```

## Store file (`{domain}.store.ts`)

1. Export the state type with an `actions` field.
2. Create the store with `create<State>()`.
3. Nest all mutations under `actions: { ...domainActions(set, get) }`.
4. Export focused action hooks.

```typescript
import { create } from 'zustand';
import { domainActions, type DomainStateActions } from './actions';

export type DomainState = {
  items: Item[];
  selectedId: string | null;
  actions: DomainStateActions;
};

export const useDomainStore = create<DomainState>((set, get) => ({
  items: [],
  selectedId: null,
  actions: {
    ...domainActions(set, get),
  },
}));

export const useDomainActions = () =>
  useDomainStore((state) => state.actions);
```

### Middleware

Add `devtools`, `persist`, or other middleware when the store benefits from debugging, hydration, or persistence. Simple stores can use plain `create`.

## Actions (`actions/{domain}.actions.ts`)

1. Export `{Domain}StateActions` with every mutation signature.
2. Export a factory: `(set, get?) => DomainStateActions`.
3. Type `set` / `get` from `StoreApi<DomainState>['setState' | 'getState']`.
4. Keep updates immutable (`set((state) => ({ ... }))` or shallow merges).
5. Call sibling actions via `get().actions.otherAction()` when needed.
6. Include `reset()` that restores initial state (do not reset the `actions` object itself).

```typescript
import type { StoreApi } from 'zustand';
import type { DomainState } from '../{domain}.store';

export type DomainStateActions = {
  setSelectedId: (id: string | null) => void;
  addItem: (item: Item) => void;
  reset: () => void;
};

type Set = StoreApi<DomainState>['setState'];
type Get = StoreApi<DomainState>['getState'];

export const domainActions = (set: Set, get: Get): DomainStateActions => ({
  setSelectedId: (id) => set({ selectedId: id }),
  addItem: (item) => set({ items: [...get().items, item] }),
  reset: () => set({ items: [], selectedId: null }),
});
```

### Derived indexes

When lookups by id are frequent, maintain a derived map alongside the list and update both in the same action:

```typescript
setNodes: (nodes) => {
  const byId = nodes.reduce(
    (acc, node) => ({ ...acc, [node.id]: node }),
    {} as Record<string, Node>
  );
  set({ nodes, byId });
},
```

### Composite / versioned actions

For stores with many sub-domains (e.g. per-entity-type config, API versions):

- Root registry in `actions/{domain}.actions.ts` composes sub-factories.
- Each sub-module exports its own actions type + factory.
- Nested shape: `actions.{feature}.v1.updateField(...)`.
- Export granular hooks from the store file:

```typescript
export const useFeatureV1Actions = () =>
  useDomainStore((state) => state.actions.feature.v1);
```

Keep shared CRUD (init, update metadata, reset) in `common.actions.ts`.

## Selectors (`selectors/`)

| Pattern | Naming | Usage |
|---------|--------|-------|
| Reactive hook | `useXxx` | Components subscribing to slices |
| Non-reactive getter | `getXxx` | Callbacks, validators, save handlers |
| Composite slice | `{domain}Selector` | Related state + actions for one feature |
| Narrow pick | `useShallow` | Derived objects/arrays to avoid re-renders |

```typescript
export const useSelectedItem = () =>
  useDomainStore((state) =>
    state.items.find((i) => i.id === state.selectedId)
  );

export const getItems = () => useDomainStore.getState().items;

export const domainSelector = (state: DomainState) => ({
  items: state.items,
  addItem: state.actions.addItem,
});
```

Group domain-specific selectors in the same file with section comments.

## Staging store pattern

Use when an edit flow needs cancel/undo without touching canonical state:

```
Authoritative store          Staging store              Validation store (optional)
───────────────────          ─────────────              ───────────────────────────
canonical entities      →    draft copy on open         errors keyed by entity id
                             edit via staging actions
Save:   validate → patch authoritative → reset staging → clear errors
Cancel: reset staging only (authoritative unchanged)
```

Typical split: one store holds the source of truth; a second holds in-progress edits; a third (optional) holds field-level validation errors.

## Component usage

- **Read**: `useDomainStore((s) => s.field)` or dedicated selector hooks.
- **Write**: `useDomainActions()` or granular action hooks — never `set` in components.
- **Outside render** (save handlers, validators): `getXxx()` via `store.getState()`.

## Testing actions

Test action factories in isolation — no React, no full store:

1. Build a mutable `state` object; mock `set` / `get`.
2. Instantiate `domainActions(set, get)` and attach to `state.actions`.
3. Assert state via `getState()` after calling actions.

See [examples.md](examples.md) for templates.

## New store checklist

```
- [ ] Create {storeRoot}/{domain}/{domain}.store.ts with State type + create()
- [ ] Add actions/{domain}.actions.ts with typed factory + reset
- [ ] Wire actions in store; export useDomainActions (+ granular hooks if nested)
- [ ] Add selectors for commonly read slices (use* + get* pairs)
- [ ] Add actions/index.ts and selectors/index.ts re-exports
- [ ] Add actions/{domain}.actions.test.ts for non-trivial logic
- [ ] Components use action hooks only; no direct setState on store
```

## Anti-patterns

- Business logic inline in the store file instead of `actions/`.
- Subscribing to the entire store when a narrow selector suffices.
- Mixing fetched/server data into Zustand when a cache layer already owns it.
- Forgetting `get()` when an action depends on current state.
- Skipping `reset()` on stores tied to route/panel/modal lifecycle.

## Additional resources

- Full templates: [examples.md](examples.md)
