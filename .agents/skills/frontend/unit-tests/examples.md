# Unit Test Examples

Generic templates. Swap `mockFn`, `mockModule`, `vi`, and `jest` for the project's test runner.

## Component test

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi, beforeEach } from 'vitest';

import { LoginForm } from './login-form';

describe('LoginForm', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('shows a validation error when email is empty', async () => {
    // Arrange
    render(<LoginForm onSubmit={vi.fn()} />);

    // Act
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    // Assert
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
  });

  it('submits credentials when the form is valid', async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'secret');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    expect(onSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'secret',
    });
  });
});
```

## Component with providers and router mock

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';

import { SettingsPage } from './settings-page';
import { renderWithProviders } from '../../test/render-with-providers';

const { navigate } = vi.hoisted(() => ({ navigate: vi.fn() }));

vi.mock('../../router', () => ({
  useNavigate: () => navigate,
}));

describe('SettingsPage', () => {
  it('navigates home after saving', async () => {
    const user = userEvent.setup();
    renderWithProviders(<SettingsPage />);

    await user.click(screen.getByRole('button', { name: /save/i }));

    expect(navigate).toHaveBeenCalledWith('/');
  });
});
```

## Hook test

```typescript
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';

import { useCounter } from './use-counter';

describe('useCounter', () => {
  it('starts at the initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it('increments the count', () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

## Pure function test

```typescript
import { describe, it, expect } from 'vitest';

import { formatCurrency } from './format-currency';

describe('formatCurrency', () => {
  it('formats a positive amount with the currency symbol', () => {
    expect(formatCurrency(1234.5, 'USD')).toBe('$1,234.50');
  });

  it('returns a zero display for null input', () => {
    expect(formatCurrency(null, 'USD')).toBe('$0.00');
  });
});
```

## Service / API layer test

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

import { UserService } from './user-service';

const get = vi.fn();

vi.mock('./http-client', () => ({
  httpClient: { get: (...args: unknown[]) => get(...args) },
}));

describe('UserService.getById', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('maps the API response to a domain user', async () => {
    get.mockResolvedValue({ data: { id: '1', full_name: 'Ada Lovelace' } });

    const user = await UserService.getById('1');

    expect(get).toHaveBeenCalledWith('/users/1');
    expect(user).toEqual({ id: '1', name: 'Ada Lovelace' });
  });

  it('throws when the request fails', async () => {
    get.mockRejectedValue(new Error('network error'));

    await expect(UserService.getById('1')).rejects.toThrow('network error');
  });
});
```

## Store action test (no render)

```typescript
import { describe, it, expect, vi } from 'vitest';
import type { StoreApi } from 'zustand';

import { cartActions } from './cart.actions';
import type { CartState } from '../cart.store';

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

  return { getState: () => state, actions };
}

describe('cartActions', () => {
  it('adds an item and indexes it by id', () => {
    const { actions, getState } = createTestCartState();
    const item = { id: 'a1', name: 'Widget', qty: 1 };

    actions.addItem(item);

    expect(getState().items).toEqual([item]);
    expect(getState().itemsById).toEqual({ a1: item });
  });
});
```

## Fixture factory

```typescript
type User = { id: string; name: string; role: 'admin' | 'member' };

export function createUser(overrides: Partial<User> = {}): User {
  return {
    id: 'user-1',
    name: 'Test User',
    role: 'member',
    ...overrides,
  };
}

// Usage in tests
const admin = createUser({ role: 'admin', name: 'Admin User' });
```

## Asserting absence (async)

```typescript
it('does not show a success message before submit', () => {
  render(<ContactForm />);
  expect(screen.queryByText(/message sent/i)).not.toBeInTheDocument();
});

it('shows a success message after submit', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);

  await user.click(screen.getByRole('button', { name: /send/i }));

  expect(await screen.findByText(/message sent/i)).toBeInTheDocument();
});
```

## Jest equivalents

| Vitest | Jest |
|--------|------|
| `vi.fn()` | `jest.fn()` |
| `vi.mock()` | `jest.mock()` |
| `vi.hoisted()` | define mocks before `jest.mock` or use `jest.requireActual` patterns |
| `vi.clearAllMocks()` | `jest.clearAllMocks()` |
| `vi.useFakeTimers()` | `jest.useFakeTimers()` |
