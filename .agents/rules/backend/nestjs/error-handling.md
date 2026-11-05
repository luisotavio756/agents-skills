# Error Handling

## Error Catalog

All application errors live in a single catalog object. This centralizes error codes, HTTP statuses, and messages — never throw raw `HttpException` directly.

### `defineError` helper

```typescript
// src/shared/errors/define-error.ts
import { HttpException } from '@nestjs/common';

interface ErrorConfig {
  status: number;
  code: string;
  message: string;
}

export function defineError(config: ErrorConfig): HttpException {
  return new HttpException(
    {
      statusCode: config.status,
      code: config.code,
      message: config.message,
    },
    config.status,
  );
}
```

### Catalog definition

```typescript
// src/shared/errors/errors-catalog.ts
import { defineError } from './define-error';

export const ErrorsCatalog = {
  Users: {
    NotFound: defineError({
      status: 404,
      code: 'USERS_NOT_FOUND',
      message: 'User not found',
    }),
    AlreadyExists: defineError({
      status: 409,
      code: 'USERS_ALREADY_EXISTS',
      message: 'User already exists',
    }),
    Unauthorized: defineError({
      status: 401,
      code: 'USERS_UNAUTHORIZED',
      message: 'Invalid credentials',
    }),
  },
  Products: {
    NotFound: defineError({
      status: 404,
      code: 'PRODUCTS_NOT_FOUND',
      message: 'Product not found',
    }),
    AlreadyExists: defineError({
      status: 409,
      code: 'PRODUCTS_ALREADY_EXISTS',
      message: 'Product already exists',
    }),
  },
} as const;
```

### Usage in services

```typescript
import { ErrorsCatalog } from '@shared/errors/errors-catalog';

async findById(id: string): Promise<FindUserResponseDto> {
  const user = await this.userRepository.findById(id);
  if (!user) throw ErrorsCatalog.Users.NotFound;
  return this.toResponseDto(user);
}

async create(dto: CreateUserDto): Promise<CreateUserResponseDto> {
  const existing = await this.userRepository.findByEmail(dto.email);
  if (existing) throw ErrorsCatalog.Users.AlreadyExists;
  // ...
}
```

## Error code convention

```
{DOMAIN}_{ERROR_TYPE}
```

| Domain | Example codes |
|--------|--------------|
| Users | `USERS_NOT_FOUND`, `USERS_ALREADY_EXISTS`, `USERS_UNAUTHORIZED` |
| Products | `PRODUCTS_NOT_FOUND`, `PRODUCTS_OUT_OF_STOCK` |
| Orders | `ORDERS_NOT_FOUND`, `ORDERS_INVALID_STATUS` |

- Domain prefix in SCREAMING_SNAKE — matches the catalog key
- Error type describes the condition, not the operation

## Global exception filter

Create in `core/filters/` and register globally. Formats both catalog errors and unexpected failures:

```typescript
// src/core/filters/global-exception.filter.ts
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    if (exception instanceof HttpException) {
      const body = exception.getResponse();
      response.status(exception.getStatus()).json({
        ...(typeof body === 'object' ? body : { message: body }),
        timestamp: new Date().toISOString(),
      });
      return;
    }

    // Unexpected — log and return 500
    console.error('[Unhandled]', exception);
    response.status(500).json({
      statusCode: 500,
      code: 'INTERNAL_SERVER_ERROR',
      message: 'Internal server error',
      timestamp: new Date().toISOString(),
    });
  }
}
```

Register in `main.ts`:

```typescript
app.useGlobalFilters(new GlobalExceptionFilter());
```

## File location

```
src/shared/errors/
├── define-error.ts      # defineError helper
└── errors-catalog.ts    # ErrorsCatalog object
```

## Principles

- **Always** throw from the catalog — never `new HttpException(...)` inline
- Throw in **services** only — never in controllers or repositories
- Repositories return `null` on not found; the service throws the catalog error
- Never expose stack traces or internal messages in production responses
- Add a new domain section to the catalog when scaffolding a new module

## Avoid

- `throw new NotFoundException('...')` — use the catalog instead
- `throw new HttpException(...)` inline — not discoverable, not standardized
- Throwing from controllers or repositories
- Error codes that describe the operation instead of the condition (`USERS_GET_FAILED` → `USERS_NOT_FOUND`)
