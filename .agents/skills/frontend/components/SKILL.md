---
name: components
description: >-
  Create or refactor React components following project conventions. Use when
  writing a new component, extracting a sub-component, or reviewing an existing
  component's props design, composition, and structure.
---

# React Component Creation

## Before writing

1. Confirm the component does **one thing** — if it does two, split it
2. Choose the right location:
   - `ui/` — generic, reusable anywhere
   - `shared/` — shared across features but not fully generic
   - `pages/<Feature>/components/` — scoped to a single page/feature
3. Define the props interface **before** the implementation — design the API first

## Props design

Co-locate types in `ComponentName.types.ts`:

```typescript
// ComponentName.types.ts
export type UserCardProps = {
  name: string;
  avatarUrl: string;
  onEdit?: () => void;
};
```

**Extend HTML props** when the component wraps a native element:

```typescript
type InputProps = React.InputHTMLAttributes<HTMLInputElement> & {
  label: string;
  error?: string;
};
```

**Use discriminated unions** for mutually exclusive states:

```typescript
type AlertProps =
  | { variant: 'info' | 'success'; message: string }
  | { variant: 'error'; message: string; onRetry: () => void };
```

**Prop conventions:**
- `children` for slot content — prefer over render props unless children need context
- Spread `...rest` onto the root element for HTML passthrough
- Forward `className` to allow composability from the outside

## Component structure

```typescript
// 1. Imports
// 2. Types (or import from .types.ts)
// 3. Component function
// 4. Tightly coupled sub-components (if small)
// 5. Module-level helpers (or move to .utils.ts if reused)

export function UserCard({ name, avatarUrl, onEdit }: UserCardProps) {
  return (
    <div className="flex items-center gap-3 rounded-lg border p-4">
      <Avatar src={avatarUrl} alt={name} />
      <span className="flex-1 truncate font-medium">{name}</span>
      {onEdit && (
        <Button variant="ghost" size="sm" onClick={onEdit}>
          Edit
        </Button>
      )}
    </div>
  );
}
```

## Render states

Every component with async data must handle all three states:

```typescript
if (isLoading) return <Skeleton className="h-24 w-full rounded-lg" />;
if (error) return <ErrorMessage message={error.message} />;
if (!items.length) return <EmptyState label="No items yet" />;
return <ItemList items={items} />;
```

## Forwarding refs

Use `forwardRef` when the parent needs direct DOM access:

```typescript
export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, ...props }, ref) => (
    <div className="flex flex-col gap-1">
      <label className="text-sm font-medium">{label}</label>
      <input
        ref={ref}
        className={cn('border rounded-md px-3 py-2', error && 'border-destructive')}
        {...props}
      />
      {error && <p className="text-sm text-destructive">{error}</p>}
    </div>
  )
);
Input.displayName = 'Input';
```

## Custom hooks

Extract into a hook when:
- The component has more than one `useEffect`
- Multiple components share the same logic
- Side-effect logic obscures the render path

```typescript
// useUserData.ts — extracted, testable in isolation
function useUserData(userId: string) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => UserService.getById(userId),
  });
  return { user: data, isLoading, error };
}
```

## Styling

- Tailwind classes only — no inline styles
- Use `cn()` (clsx + tailwind-merge) for conditional classes
- Prefer design system tokens (`bg-background`, `text-foreground`) over raw colors
- Mobile-first: base class = mobile, add `md:` / `lg:` for wider breakpoints

```typescript
import { cn } from '@/lib/utils';

<div className={cn('rounded-md border p-4', isSelected && 'border-primary bg-primary/5')} />
```

## Accessibility

- Interactive elements: `<button>` for actions, `<a>` for navigation — never `<div onClick>`
- Icon-only buttons: `aria-label` required
- Form inputs: associated `<label>` via `htmlFor` or wrapping
- Focus visible: `focus-visible:ring-2 focus-visible:ring-ring` on all interactive elements
- Decorative icons: `aria-hidden="true"`
- Images: `alt` text (empty string if decorative)

## Deliverable checklist

```
- [ ] Folder: ComponentName/ with index.ts, ComponentName.tsx, ComponentName.types.ts
- [ ] index.ts is a clean re-export only (no logic)
- [ ] Props typed and exported from .types.ts
- [ ] All render states handled: loading, error, empty
- [ ] Accessible: ARIA labels, keyboard, focus-visible
- [ ] Tailwind only — no inline styles
- [ ] Under 200 lines
- [ ] ComponentName.test.tsx created alongside the component
```
