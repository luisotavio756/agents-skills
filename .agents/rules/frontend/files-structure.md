# File Structure & Organization

For naming conventions (file names, component names, etc.) see [casing.md](casing.md).

## Components folder

```text
src/components/
├── index.ts        # Re-exports public components
├── ui/             # Generic UI primitives (Button, Toast, Modal…)
└── shared/         # Cross-feature components (Avatar, Breadcrumb…)
```

Feature-scoped components live inside the page/feature folder, not here.

## Component directory layout

Every component — whether in `ui/`, `shared/`, or a page folder — follows the same structure:

```text
ComponentName/
├── index.ts                    # Re-export only; no logic here
├── ComponentName.tsx           # Main component (< 200 lines)
├── ComponentName.test.tsx      # Unit tests
├── ComponentName.types.ts      # TypeScript definitions
├── ComponentName.utils.ts      # Pure helpers (only if needed)
├── ComponentName.stories.ts    # Storybook (if applicable)
└── components/                 # Sub-components scoped to this component
    ├── ChildA/
    └── ChildB/
```

## Page folder layout

```text
src/pages/FeatureName/
├── index.ts
├── FeatureName.tsx             # Page entry component
├── FeatureName.test.tsx
├── FeatureName.types.ts
└── components/                 # Components used only by this page
    └── SomeWidget/
```

## Rules

- `index.ts` is a clean re-export only — no JSX, no logic
- Sub-components only go in `components/` if they are not reused outside the parent
- If a sub-component is needed in two places, move it up to `shared/`
- Keep test files colocated with source, not in a top-level `__tests__/` directory
