# DTOs & Validation

## Global validation pipe

Register once in `main.ts`:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,               // Strip unknown properties
    forbidNonWhitelisted: true,    // Throw on unknown properties
    transform: true,               // Auto-transform to DTO class instances
    transformOptions: { enableImplicitConversion: true },
  })
);
```

## Input DTOs

```typescript
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  name: string;

  @IsEmail()
  email: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}
```

**Rules:**
- `@IsOptional()` must be the first decorator on optional fields
- `@IsString()` alone allows empty string — pair with `@IsNotEmpty()` when required
- Use `@Type(() => Number)` for numeric query/path params
- Use `@Transform(({ value }) => value.trim())` for sanitization
- Nested objects: `@ValidateNested()` + `@Type(() => AddressDto)`

## Composition DTOs

Avoid duplication with mapped types:

```typescript
export class UpdateUserDto extends PartialType(CreateUserDto) {}
export class UserQueryDto extends PickType(CreateUserDto, ['email'] as const) {}
export class UserWithoutPasswordDto extends OmitType(CreateUserDto, ['password'] as const) {}
```

## Output DTOs

Never return raw entities — always map to a response DTO. Each use-case service has its own response DTO co-located in its `dto/` folder:

```typescript
// services/create-user/dto/create-user-response.dto.ts
export class CreateUserResponseDto {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  // No passwordHash, no internal fields
}
```

Map from entity inside the service's private `toResponseDto()` method:

```typescript
private toResponseDto(user: User): CreateUserResponseDto {
  return {
    id: user.id,
    name: user.name,
    email: user.email,
    createdAt: user.createdAt,
  };
}
```

## Env validation with Zod

Validate all environment variables at startup — fail fast on missing/invalid config:

```typescript
// config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);
export type Env = z.infer<typeof envSchema>;
```

## Avoid

- Returning entity objects directly from controllers
- Validating input inside service methods (use DTOs + GlobalValidationPipe)
- `@IsString()` without `@IsNotEmpty()` on required string fields
- `@IsOptional()` after other decorators
- Duplicating DTO fields instead of using `PartialType` / `OmitType`
