# Swagger Documentation

## Plugin setup (JSDoc introspection)

Enable the `@nestjs/swagger` CLI plugin in `nest-cli.json` so JSDoc comments become API docs automatically — no `@ApiOperation` needed:

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "introspectComments": true
        }
      }
    ]
  }
}
```

With this, the JSDoc above each controller method is used as the Swagger `summary` and `description`.

## Documenting routes with JSDoc

Write JSDoc above the method — never use `@ApiOperation`:

```typescript
/**
 * Creates a new user account.
 *
 * Validates the email uniqueness before persisting.
 * Returns the created user data on success.
 */
@Post()
@SetApiCreatedResponse(CreateUserResponseDto)
@SetApiConflictResponse()
@SetApiBadRequestResponse()
create(@Body() dto: CreateUserDto): Promise<CreateUserResponseDto> {
  return this.createUserService.execute(dto);
}
```

The first line becomes the `summary`; subsequent lines become the `description`.

## Custom response decorators

Live in `src/shared/decorators/api-response.decorators.ts`. Import and use in controllers — never repeat raw Swagger decorators inline.

### Implementation

```typescript
// src/shared/decorators/api-response.decorators.ts
import { applyDecorators, Type } from '@nestjs/common';
import {
  ApiOkResponse,
  ApiCreatedResponse,
  ApiNoContentResponse,
  ApiBadRequestResponse,
  ApiUnauthorizedResponse,
  ApiForbiddenResponse,
  ApiNotFoundResponse,
  ApiConflictResponse,
  ApiUnprocessableEntityResponse,
} from '@nestjs/swagger';

const errorSchema = (statusCode: number, code: string, message: string) => ({
  schema: {
    example: {
      statusCode,
      code,
      message,
      timestamp: '2024-01-01T00:00:00.000Z',
    },
  },
});

export function SetApiOkResponse<T>(dto: Type<T>) {
  return applyDecorators(ApiOkResponse({ type: dto }));
}

export function SetApiCreatedResponse<T>(dto: Type<T>) {
  return applyDecorators(ApiCreatedResponse({ type: dto }));
}

export function SetApiNoContentResponse() {
  return applyDecorators(ApiNoContentResponse({ description: 'No content' }));
}

export function SetApiBadRequestResponse() {
  return applyDecorators(
    ApiBadRequestResponse({
      description: 'Bad request',
      ...errorSchema(400, 'BAD_REQUEST', 'Validation failed'),
    }),
  );
}

export function SetApiUnauthorizedResponse() {
  return applyDecorators(
    ApiUnauthorizedResponse({
      description: 'Unauthorized',
      ...errorSchema(401, 'UNAUTHORIZED', 'Invalid or missing credentials'),
    }),
  );
}

export function SetApiForbiddenResponse() {
  return applyDecorators(
    ApiForbiddenResponse({
      description: 'Forbidden',
      ...errorSchema(403, 'FORBIDDEN', 'Insufficient permissions'),
    }),
  );
}

export function SetApiNotFoundResponse(entity = 'Resource') {
  return applyDecorators(
    ApiNotFoundResponse({
      description: 'Not found',
      ...errorSchema(404, 'NOT_FOUND', `${entity} not found`),
    }),
  );
}

export function SetApiConflictResponse(description = 'Resource already exists') {
  return applyDecorators(
    ApiConflictResponse({
      description: 'Conflict',
      ...errorSchema(409, 'CONFLICT', description),
    }),
  );
}

export function SetApiUnprocessableEntityResponse() {
  return applyDecorators(
    ApiUnprocessableEntityResponse({
      description: 'Unprocessable entity',
      ...errorSchema(422, 'UNPROCESSABLE_ENTITY', 'Semantic validation failed'),
    }),
  );
}
```

### Barrel export

```typescript
// src/shared/decorators/index.ts
export * from './api-response.decorators';
```

## Controller usage

```typescript
import {
  SetApiCreatedResponse,
  SetApiOkResponse,
  SetApiNoContentResponse,
  SetApiBadRequestResponse,
  SetApiUnauthorizedResponse,
  SetApiNotFoundResponse,
  SetApiConflictResponse,
} from '@shared/decorators';

@Controller('users')
@SetApiUnauthorizedResponse()      // applied to all routes in this controller
export class UserController {
  /**
   * Creates a new user account.
   */
  @Post()
  @SetApiCreatedResponse(CreateUserResponseDto)
  @SetApiBadRequestResponse()
  @SetApiConflictResponse('Email already in use')
  create(@Body() dto: CreateUserDto): Promise<CreateUserResponseDto> {
    return this.createUserService.execute(dto);
  }

  /**
   * Returns a user by ID.
   */
  @Get(':id')
  @SetApiOkResponse(FindUserResponseDto)
  @SetApiNotFoundResponse('User')
  findById(@Param('id') id: string): Promise<FindUserResponseDto> {
    return this.findUserService.findById(id);
  }

  /**
   * Deletes a user by ID.
   */
  @Delete(':id')
  @SetApiNoContentResponse()
  @SetApiNotFoundResponse('User')
  delete(@Param('id') id: string): Promise<void> {
    return this.deleteUserService.execute(id);
  }
}
```

## File location

```
src/shared/decorators/
├── api-response.decorators.ts
└── index.ts
```

## DTO properties

With `introspectComments: true`, JSDoc on DTO properties generates `@ApiProperty` descriptions automatically — no need to annotate each field manually:

```typescript
export class CreateUserDto {
  /** Full name of the user */
  @IsString()
  @IsNotEmpty()
  name: string;

  /** Primary email address */
  @IsEmail()
  email: string;
}
```

## Avoid

- `@ApiOperation({ summary: '...' })` — use JSDoc instead
- Raw `@ApiOkResponse`, `@ApiNotFoundResponse`, etc. inline in controllers — use the custom decorators
- Undocumented controller methods — every public route needs a JSDoc summary
- Inconsistent error response shapes — always use the `errorSchema` helper format
