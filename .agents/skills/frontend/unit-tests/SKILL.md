---
name: unit-tests
description: >-
  Write clear, maintainable unit tests using the AAA pattern, behavior-focused
  names, and minimal mocking. Use when creating or updating tests, writing
  *.test.ts(x), mocking dependencies, or when the user asks to test a component,
  hook, store, service, or utility.
---

# Unit Tests

Adapt to the project's test runner, libraries, and conventions (Vitest, Jest, Testing Library, Mocha, etc.). When project-specific rules or shared helpers exist, follow those first — this skill covers universal structure and habits.

## Principles

1. **Test behavior, not implementation** — assert what the user or caller sees/experiences, not private internals.
2. **One logical scenario per test** — split when a name needs "and" to describe multiple outcomes.
3. **Keep tests simple** — happy path plus meaningful edge cases; skip exhaustive matrices unless asked.
4. **Mock external boundaries** — network, filesystem, router, global stores, timers, third-party SDKs. Don't mock the unit under test.
5. **Colocate tests** — `{name}.test.ts(x)` next to source, unless the project uses `__tests__/` or `tests/`.

## AAA pattern

Structure every test as **Arrange → Act → Assert**. Add section comments when setup is non-trivial; skip them for short 3-line tests.

```typescript
it('submits credentials when the form is valid', async () => {
  // Arrange
  const onSubmit = mockFn();
  render(<LoginForm onSubmit={onSubmit} />);
  await user.type(screen.getByLabelText(/email/i), 'user@example.com');
  await user.type(screen.getByLabelText(/password/i), 'secret');

  // Act
  await user.click(screen.getByRole('button', { name: /sign in/i }));

  // Assert
  expect(onSubmit).toHaveBeenCalledWith({
    email: 'user@example.com',
    password: 'secret',
  });
});
```

Use `mockFn()` as a stand-in for `vi.fn()` (Vitest) or `jest.fn()` (Jest).

## Naming

Describe the scenario in plain language. Prefer behavior over method or file names.

| Good | Avoid |
|------|-------|
| `shows an error when email is empty` | `test email validation` |
| `disables the save button while loading` | `handles loading state` |
| `returns an empty list when input is null` | `convert works` |

**`describe` blocks:** name the unit under test (`LoginForm`, `formatCurrency`, `UserService.create`).

**`it` blocks:** state the expected outcome given a condition. Some projects require phrasing like **"should be able to …"** — follow that convention when present.

## File structure

```typescript
// 1. Imports
// 2. Module mocks (top level)
// 3. Local fixtures / helpers (or imports from shared test utils)
// 4. describe → beforeEach/afterEach → it blocks

describe('ComponentOrModule', () => {
  beforeEach(() => {
    clearAllMocks();
  });

  describe('when the user is authenticated', () => {
    it('renders the dashboard link', () => { /* AAA */ });
  });
});
```

Group related cases with nested `describe` blocks (by feature, state, or input variant).

## What to test

| Layer | Focus |
|-------|-------|
| **UI components** | Key content, interactions, conditional visibility, disabled/loading states, errors |
| **Hooks** | Return values and side effects; use `renderHook` from your testing library |
| **Pure functions** | Input → output; edge cases (null, empty, boundaries) |
| **Services / API layers** | Request shape, response mapping, error handling (mock the HTTP client) |
| **State / actions** | State before and after an action; no UI render needed |

**Skip unless asked:** private helpers, library internals, snapshot-only tests, duplicate coverage of the same path.

## Mocking

- Mock at the **boundary** (module import, API client, router), not deep inside the unit.
- Reset mocks in `beforeEach` (`clearAllMocks()`).
- Use **hoisted** mock functions when the mock factory references them (Vitest `vi.hoisted`; Jest hoisting patterns).
- Prefer **partial mocks** (`importOriginal` / `requireActual`) when only one export needs stubbing.
- For heavy child components, stub with a minimal stand-in that records props or exposes a test id.

```typescript
const { navigate } = hoisted(() => ({ navigate: mockFn() }));

mockModule('./router', () => ({
  useNavigate: () => navigate,
}));
```

Replace `hoisted`, `mockFn`, and `mockModule` with the runner's API (`vi.*` or `jest.*`).

## UI query priority (Testing Library)

Query in order of accessibility:

1. **`getByRole`** — buttons, links, headings, inputs (with `name`)
2. **`getByLabelText`** — form fields
3. **`getByPlaceholderText`** / **`getByText`** — when role/label aren't available
4. **`getByTestId`** — last resort; add `data-testid` only when needed
5. **`queryBy*`** — asserting an element is **not** present
6. **`findBy*`** — async appearance (`await findByRole(...)`)

Avoid CSS selectors, DOM traversal, and implementation details. If project rules prefer test ids over roles, follow the project.

## Shared test utilities

When the same setup repeats, extract to a project test folder (e.g. `test/utils/`, `tests/helpers/`):

- **Fixtures** — factory functions with defaults and overrides (`createUser({ role: 'admin' })`)
- **Render helpers** — wrap components with providers (router, theme, data-fetching client)
- **Shared mocks** — side-effect imports for modules many tests depend on

Don't extract one-off helpers used in a single file.

## Async and cleanup

- Prefer `userEvent.setup()` with `async/await` over raw `fireEvent` unless necessary.
- Use `waitFor` / `findBy*` for async UI updates.
- Restore globals in `afterEach`: env stubs, `window.location`, fake timers.

## Checklist

```
- [ ] Test file follows project colocation convention
- [ ] External dependencies mocked; unit under test is real
- [ ] AAA structure — readable Arrange / Act / Assert
- [ ] Name describes behavior and expected outcome
- [ ] Queries follow project priority (role/label vs test id)
- [ ] One scenario per it block
- [ ] Mocks cleared between tests
- [ ] Test run passes for the changed file
```

## Additional resources

- Layer-specific templates: [examples.md](examples.md)
