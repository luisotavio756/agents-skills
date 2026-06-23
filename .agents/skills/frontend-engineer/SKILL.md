# Rules

Act as a senior frontend engineer.

## Stack:

- React
- TypeScript
- Tailwind
- Shadcn

Generate production-quality code.

Do not introduce unnecessary abstractions.

## Creating new components:

- Prefer creating a folder, instead using a file directly inside some folder under components.
- Prefer using kebab case for composite names.

### UI components example:

- src/components/ui/
    button/
        - button.tsx
        - button.stories.tsx (ask the user)
        - button.test.tsx (if asked)
        - index.ts (exports the main component)
    ...
    index.ts (exports all the components under ui)

### Shared components example:

- src/components/shared/
    availability-badge/
        - availability-badge.tsx
        - availability-badge.stories.tsx (ask the user)
        - availability-badge.test.tsx (if asked)
        - index.ts (exports the main component)
    ...
    index.ts (exports all the components under shared)

### Feature components example:

- src/components/features/
    dashboard/
        dashboard-kpi-card/
            - dashboard-kpi-card.tsx
            - dashboard-kpi-card.stories.tsx (ask the user)
            - dashboard-kpi-card.test.tsx (if asked)
            - index.ts (exports the main component)
        ...
        index.ts (exports all the components under dashboard)
    ...
    index.ts (exports all the components under features)

If the component/page is splitted into small parts, prefer using the base name plus an additional context. Examples:

- src/pages/dashboard/
    - dashboard.tsx
    - dashboard-header.tsx
    - index.ts

- src/pages/some-page-with-a-compose-name/
    - some-page-with-a-compose-name.tsx
    - some-page-with-a-compose-name.header.tsx
    - some-page-with-a-compose-name.header-something.tsx
    - index.ts





