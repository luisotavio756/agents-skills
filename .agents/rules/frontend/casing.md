# Casing & Naming Conventions

## Files

| Type | Convention | Example |
|------|-----------|---------|
| React component | PascalCase | `UserProfile.tsx` |
| Utility / helper | kebab-case | `format-date.ts` |
| Hook | camelCase | `useAuthSession.ts` |
| Type definitions | PascalCase suffix | `UserProfile.types.ts` |
| Test file | same as source + `.test` | `UserProfile.test.tsx` |
| Constants / config | kebab-case | `query-keys.ts` |
| Store | kebab-case | `auth-store.ts` |

## Code

| Type | Convention | Example |
|------|-----------|---------|
| Component | PascalCase | `UserProfile` |
| Function | camelCase | `formatCurrency` |
| Hook | `use` prefix | `useAuthSession` |
| Variable | camelCase | `isLoading` |
| Boolean | `is`/`has`/`can` prefix | `isOpen`, `hasError`, `canSubmit` |
| Module-level constant | SCREAMING_SNAKE | `MAX_RETRIES` |
| Type / Interface | PascalCase | `UserProfileProps` |
| Enum member | PascalCase | `Status.Active` |
| Event handler prop | `on` prefix | `onSubmit`, `onChange` |
| Internal handler | `handle` prefix | `handleSubmit`, `handleChange` |

## Props

- Omit value for boolean props that default to `true`: `<Button disabled />`
- Callbacks in the component API: `on` prefix (`onSubmit`)
- Internal implementations: `handle` prefix (`handleSubmit`)

## Avoid

- Abbreviations that aren't universally obvious (`usr`, `cfg`, `btn`)
- Single-letter names outside short loop indices or math
- Hungarian notation (`strName`, `bIsValid`)
- Generic names (`data`, `info`, `item`, `obj`) — be specific
