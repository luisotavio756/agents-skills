# Zustand Store Examples

Generic templates illustrating the conventions. Replace `{domain}`, types, and paths to match the target project.

## Simple store + actions + selectors

**`cart.store.ts`**

```typescript
import { create } from 'zustand';
import { cartActions, type CartStateActions } from './actions';

export type CartState = {
  items: CartItem[];
  itemsById: Record<string, CartItem>;
  actions: CartStateActions;
};

export const useCartStore = create<CartState>((set, get) => ({
  items: [],
  itemsById: {},
  actions: {
    ...cartActions(set, get),
  },
}));

export const useCartActions = () => useCartStore((state) => state.actions);
```

**`actions/cart.actions.ts`**

```typescript
import type { StoreApi } from 'zustand';
import type { CartState } from '../cart.store';

export type CartStateActions = {
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  reset: () => void;
};

type Set = StoreApi<CartState>['setState'];
type Get = StoreApi<CartState>['getState'];

export const cartActions = (set: Set, get: Get): CartStateActions => ({
  addItem: (item) => {
    const items = [...get().items, item];
    set({
      items,
      itemsById: { ...get().itemsById, [item.id]: item },
    });
  },
  removeItem: (id) => {
    const { [id]: _, ...itemsById } = get().itemsById;
    set({
      items: get().items.filter((i) => i.id !== id),
      itemsById,
    });
  },
  reset: () => set({ items: [], itemsById: {} }),
});
```

**`selectors/cart.selectors.ts`**

```typescript
import { useCartStore, type CartState } from '../cart.store';

export const useCartItemCount = () =>
  useCartStore((state) => state.items.length);

export const getCartItem = (id: string) =>
  useCartStore.getState().itemsById[id];

export const cartSelector = (state: CartState) => ({
  items: state.items,
  addItem: state.actions.addItem,
  removeItem: state.actions.removeItem,
});
```

## Staging store with devtools + nested actions

Use when editing entity settings in a panel/modal with save/cancel.

**`entity-draft.store.ts`**

```typescript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';
import { entityDraftActions, type EntityDraftActions } from './actions';

export type EntityDraftState = {
  draftById: Record<string, { name: string; settings: EntitySettings }>;
  actions: EntityDraftActions;
};

export const useEntityDraftStore = create<EntityDraftState>()(
  devtools((set) => ({
    draftById: {},
    actions: { ...entityDraftActions(set) },
  }))
);

export const useEntityDraftActions = () =>
  useEntityDraftStore((state) => state.actions);

export const useProfileV1DraftActions = () =>
  useEntityDraftStore((state) => state.actions.profile.v1);
```

**`actions/entity-draft.actions.ts`** (registry)

```typescript
import type { StoreApi } from 'zustand';
import type { EntityDraftState } from '../entity-draft.store';
import { commonActions, type EntityDraftCommonActions } from './common.actions';
import { profileV1Actions, type ProfileV1DraftActions } from './profile';

export type EntityDraftActions = {
  common: EntityDraftCommonActions;
  profile: { v1: ProfileV1DraftActions };
  reset: () => void;
};

type Set = StoreApi<EntityDraftState>['setState'];

export const entityDraftActions = (set: Set): EntityDraftActions => ({
  common: commonActions(set),
  profile: { v1: profileV1Actions(set) },
  reset: () => set({ draftById: {} }),
});
```

**`actions/common.actions.ts`**

```typescript
export type EntityDraftCommonActions = {
  initDraft: (id: string, name: string, settings: EntitySettings) => void;
  updateName: (id: string, name: string) => void;
  reset: () => void;
};

export const commonActions = (set: Set): EntityDraftCommonActions => ({
  initDraft: (id, name, settings) =>
    set((state) => ({
      draftById: { ...state.draftById, [id]: { name, settings } },
    })),
  updateName: (id, name) =>
    set((state) => ({
      draftById: {
        ...state.draftById,
        [id]: { ...state.draftById[id], name },
      },
    })),
  reset: () => set({ draftById: {} }),
});
```

**`actions/profile/v1.actions.ts`**

```typescript
export type ProfileV1DraftActions = {
  updateSetting: <K extends keyof EntitySettings>(
    id: string,
    field: K,
    value: EntitySettings[K]
  ) => void;
};

export const profileV1Actions = (set: Set): ProfileV1DraftActions => ({
  updateSetting: (id, field, value) =>
    set((state) => ({
      draftById: {
        ...state.draftById,
        [id]: {
          ...state.draftById[id],
          settings: { ...state.draftById[id].settings, [field]: value },
        },
      },
    })),
});
```

**`selectors/entity-draft.selectors.ts`**

```typescript
export const useDraftSettings = <T extends EntitySettings>(id: string): T =>
  useEntityDraftStore((state) => state.draftById[id]?.settings as T);

export const getDraftSettings = <T extends EntitySettings>(id: string): T =>
  useEntityDraftStore.getState().draftById[id]?.settings as T;
```

## Validation / error store with shallow selectors

**`form-errors.store.ts`**

```typescript
export type FormErrorsState = {
  errors: Record<string, Record<string, unknown>>;
  actions: FormErrorsActions;
};
```

**`selectors/form-errors.selectors.ts`**

```typescript
import { useShallow } from 'zustand/react/shallow';
import { useFormErrorsStore } from '../form-errors.store';

export const useFormErrors = <TErrors>(
  formId: string,
  pick: (errors: TErrors | undefined) => TErrors
) =>
  useFormErrorsStore(
    useShallow((state) => pick(state.errors[formId] as TErrors | undefined))
  );

export const useFormFieldError = <T = string>(formId: string, field: string) =>
  useFormErrorsStore((state) => state.errors[formId]?.[field] as T | undefined);

export const getFormErrors = <T = Record<string, unknown>>(formId: string) =>
  useFormErrorsStore.getState().errors[formId] as T | undefined;
```

## Action test harness

Works with Vitest or Jest — swap `vi.fn` for `jest.fn` as needed.

```typescript
function createTestCartState(overrides?: Partial<CartState>) {
  let state: CartState = {
    items: [],
    itemsById: {},
    actions: {} as CartState['actions'],
    ...overrides,
  };

  const set = vi.fn((partial: Partial<CartState>) => {
    state = { ...state, ...partial };
  });
  const get = () => state;

  const actions = cartActions(
    set as unknown as StoreApi<CartState>['setState'],
    get
  );
  state = { ...state, actions };

  return { getState: () => state, actions, set };
}

it('should add item and index it by id', () => {
  const { actions, getState } = createTestCartState();
  const item = { id: 'a1', name: 'Widget', qty: 1 };

  actions.addItem(item);

  expect(getState().items).toEqual([item]);
  expect(getState().itemsById).toEqual({ a1: item });
});
```

## Multi-store coordination (save/cancel flow)

```
entities.store (canonical)     entity-draft.store (staging)     form-errors.store
─────────────────────────      ────────────────────────────     ─────────────────
entities[], selectedId    →    draftById (copy on panel open)   errors[entityId]
                               edit via draft actions
Save:   validate → patch entities → reset draft → clear errors
Cancel: reset draft only
```

Cross-action call example (action A delegates to action B):

```typescript
selectEntity: (entity) => {
  get().actions.clearTransientUi();
  set({ selectedId: entity.id, panelOpen: true });
},
```
