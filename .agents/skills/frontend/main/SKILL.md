---
name: main
description: >-
  General frontend engineering skill. Use when building a new UI feature,
  implementing a page or screen, or making architectural UI decisions. Covers
  approach, decomposition, state placement, and which sub-skills to reach for.
---

# Frontend Engineering

## Before writing code

1. **Understand the goal** — what user problem does this solve? What is the success state?
2. **Map the data** — what data is needed, where does it come from (API, store, props)?
3. **Decompose into components** — sketch the component tree before writing JSX
4. **Pick the right sub-skill** — see table below

## Sub-skills

| Task | Skill | Path |
|------|-------|------|
| Creating or refactoring a component | `components` | `.agents/skills/frontend/components/SKILL.md` |
| Adding or modifying a Zustand store | `zustand-store` | `.agents/skills/frontend/zustand-store/SKILL.md` |
| Writing or updating tests | `unit-tests` | `.agents/skills/frontend/unit-tests/SKILL.md` |
| Fetching data, queries, mutations, cache | `data-fetching` | `.agents/skills/frontend/data-fetching/SKILL.md` |

## Approach to a feature

1. Identify the page entry point and its route
2. Map state: local (`useState`) vs shared (Zustand) vs server (React Query / SWR)
3. Sketch the component hierarchy top-down; identify where shared components live
4. Build leaf components first, compose upward
5. Extract repeated logic into custom hooks

## State placement

| State type | Where |
|-----------|-------|
| UI-only (open/closed, selected tab) | `useState` |
| Shared UI state across components | Zustand store |
| Server / cached data | React Query or SWR |
| Form state | `react-hook-form` or uncontrolled |
| URL-driven (filters, pagination, tabs) | Query params (`nuqs` or router) |

## Design defaults

- Mobile layout first — desktop adapts via `md:` / `lg:` breakpoints
- Loading, error, and empty states for every async operation
- Keyboard navigable; accessible by default (see `accessibility.md` rules)
- Dark mode compatible via Tailwind `dark:` variants and CSS tokens

## Component sizing signals

| Signal | Action |
|--------|--------|
| File over 200 lines | Extract sub-component or hook |
| More than 3 related props | Define a sub-type |
| More than one `useEffect` | Extract into a custom hook |
| Same JSX block repeated > 2× | Extract into a component |

## Deliverable checklist

```
- [ ] Component follows file structure convention (see files-structure.md, casing.md)
- [ ] All async states handled: loading, error, empty
- [ ] Accessible: ARIA, keyboard, focus-visible
- [ ] Mobile layout works
- [ ] Types defined in *.types.ts
- [ ] Tests written or updated (unit-tests skill)
```
