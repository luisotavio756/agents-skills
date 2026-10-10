# Testing

For detailed patterns, AAA structure, mocking recipes, and query priority, invoke the **`unit-tests` skill** (`.agents/skills/frontend/unit-tests`).

## Philosophy

- Test **behavior**, not implementation — assert what the user sees, not internals
- One logical scenario per `it` block — split when the name needs "and"
- Mock at the **boundary**: network, router, global stores, third-party SDKs
- Colocate tests: `ComponentName.test.tsx` next to source

## What to test

| Layer | Focus |
|-------|-------|
| UI components | Key interactions, conditional rendering, error/loading states |
| Hooks | Return values and side effects (`renderHook`) |
| Pure functions | Input → output; edge cases (null, empty, boundaries) |
| Services | Request shape, response mapping, error handling (mock HTTP client) |
| Stores | State before and after an action; no UI render needed |

**Skip:** private helpers, library internals, snapshot-only tests, paths already covered elsewhere.

## Tools (adapt to project)

- Runner: **Vitest** (preferred) or Jest
- UI: **React Testing Library**
- Async interactions: `userEvent.setup()` + `await`

## File structure

```text
describe('ComponentName', () => {
  beforeEach(() => clearAllMocks());

  describe('when user is authenticated', () => {
    it('renders the dashboard link', () => { /* AAA */ });
  });
});
```

## Query priority (Testing Library)

1. `getByRole` — buttons, links, headings, inputs
2. `getByLabelText` — form fields
3. `getByText` / `getByPlaceholderText` — when role/label aren't available
4. `getByTestId` — last resort only
