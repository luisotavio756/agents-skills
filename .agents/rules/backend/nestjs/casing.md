# Casing & Naming Conventions

## Files (all kebab-case)

| Type | Pattern | Example |
|------|---------|---------|
| Module | `{domain}.module.ts` | `user.module.ts` |
| Controller | `{domain}.controller.ts` | `user.controller.ts` |
| Service | `{action}-{domain}.service.ts` | `create-user.service.ts` |
| Repository | `{domain}.repository.ts` | `user.repository.ts` |
| Entity | `{domain}.entity.ts` | `user.entity.ts` |
| Input DTO | `{action}-{domain}.dto.ts` | `create-user.dto.ts` |
| Output DTO | `{action}-{domain}-response.dto.ts` | `create-user-response.dto.ts` |
| Types | `{domain}.types.ts` | `user.types.ts` |
| Constants | `{domain}.constants.ts` | `user.constants.ts` |
| Guard | `{name}.guard.ts` | `jwt-auth.guard.ts` |
| Interceptor | `{name}.interceptor.ts` | `logging.interceptor.ts` |
| Pipe | `{name}.pipe.ts` | `parse-uuid.pipe.ts` |
| Filter | `{name}.filter.ts` | `http-exception.filter.ts` |
| Decorator | `{name}.decorator.ts` | `current-user.decorator.ts` |
| Unit spec | `{name}.service.spec.ts` | `user.service.spec.ts` |
| E2E spec | `{name}.e2e-spec.ts` | `user.e2e-spec.ts` |

## Classes (all PascalCase with suffix)

| Type | Suffix | Example |
|------|--------|---------|
| Module | `Module` | `UserModule` |
| Controller | `Controller` | `UserController` |
| Service | `{Action}{Domain}Service` | `CreateUserService`, `FindUserService` |
| Repository | `Repository` | `UserRepository` |
| Entity | _(none)_ | `User` |
| Input DTO | `Dto` | `CreateUserDto` |
| Output DTO | `Dto` | `CreateUserResponseDto`, `FindUserResponseDto` |
| Guard | `Guard` | `JwtAuthGuard` |
| Interceptor | `Interceptor` | `LoggingInterceptor` |
| Pipe | `Pipe` | `ParseUuidPipe` |
| Filter | `Filter` | `HttpExceptionFilter` |
| Decorator | _(none)_ | `CurrentUser` |
| Custom exception | _(extends base)_ | `UserNotFoundException` |

## Methods and variables

- Methods and variables: camelCase, start with a verb
- Boolean: `isActive`, `hasPermission`, `canDelete`
- Void methods: `createUser`, `updateProfile`, `deleteById`, `sendEmail`
- Private helpers: camelCase with descriptive name

## Other

- Environment variables: `SCREAMING_SNAKE_CASE` (`DATABASE_URL`, `JWT_SECRET`)
- Constants: `SCREAMING_SNAKE_CASE` in a dedicated `constants.ts` file
- No abbreviations — `getUserById` not `getUsrById`
- Exception: well-known ones — `id`, `dto`, `req`, `res`, `ctx`, `err`
