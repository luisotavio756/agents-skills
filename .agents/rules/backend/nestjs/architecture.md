# Architecture & File Structure

## Module layout

```
src/
├── main.ts
├── app.module.ts
├── core/                          # Global NestJS artifacts
│   ├── filters/                   # Global exception filters
│   ├── guards/                    # Auth, roles guards
│   ├── interceptors/              # Logging, response transform
│   ├── pipes/                     # Global validation pipe
│   └── core.module.ts
├── shared/                        # Cross-module services and utilities
│   └── shared.module.ts
└── {domain}/                      # One folder per business domain
    ├── {domain}.module.ts
    ├── {domain}.controller.ts
    ├── {domain}.repository.ts
    ├── entities/
    │   └── {domain}.entity.ts
    ├── types/
    │   └── {domain}.types.ts
    │   └── {domain}-related-context.types.ts
    ├── constants/
    │   └── {domain}.constants.ts
    │   └── {domain}-related-context.constants.ts
    └── services/
        ├── create-{domain}/
        │   ├── create-{domain}.service.ts
        │   └── dto/
        │       ├── create-{domain}.dto.ts
        │       └── create-{domain}-response.dto.ts
        ├── update-{domain}/
        │   ├── update-{domain}.service.ts
        │   └── dto/
        │       ├── update-{domain}.dto.ts
        │       └── update-{domain}-response.dto.ts
        ├── delete-{domain}/
        │   └── delete-{domain}.service.ts
        └── find-{domain}/
            ├── find-{domain}.service.ts
            └── dto/
                └── find-{domain}-response.dto.ts
```

## Use-case services

Each operation (`create`, `update`, `delete`, `find`, etc.) is its own service with a **single responsibility**. DTOs are co-located inside the service folder — they belong to the use case, not to the domain as a whole.

- One service per use case — never a god service with all CRUD methods
- Each service exposes a single public method, conventionally `execute()`
- DTOs live inside the service folder: input and output are scoped to the operation

## Module definition

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([DomainEntity])],
  controllers: [DomainController],
  providers: [
    DomainRepository,
    CreateDomainService,
    UpdateDomainService,
    DeleteDomainService,
    FindDomainService,
  ],
  exports: [FindDomainService], // Export only what other modules need
})
export class DomainModule {}
```

## Layer responsibilities

### Controller
- Defines routes and HTTP methods
- Applies guards, interceptors, and decorators
- Binds and transforms input via DTOs
- Injects the specific service for each route — one service per endpoint
- Returns the response shape — no logic

```typescript
@Post()
@HttpCode(HttpStatus.CREATED)
create(@Body() dto: CreateDomainDto): Promise<CreateDomainResponseDto> {
  return this.createDomainService.execute(dto);
}
```

### Service (use case)
- Owns the business logic for one specific operation
- Receives the DTO, orchestrates repository calls, returns the response DTO
- Throws HTTP exceptions on failure
- No direct HTTP concerns, no `@Req()`, no `Response` object

```typescript
@Injectable()
export class CreateDomainService {
  constructor(private readonly domainRepository: DomainRepository) {}

  async execute(dto: CreateDomainDto): Promise<CreateDomainResponseDto> {
    const domain = await this.domainRepository.save(dto);
    return this.toResponseDto(domain);
  }

  private toResponseDto(domain: Domain): CreateDomainResponseDto {
    return { id: domain.id, name: domain.name, createdAt: domain.createdAt };
  }
}
```

### Repository
- Data access only — queries, persistence, transactions
- Returns `null` on not found — never throws
- No business logic

```typescript
async findById(id: string): Promise<Domain | null> {
  return this.repo.findOne({ where: { id } });
}
```

## Dependency rules

- No layer skips another — controllers never touch repositories
- Services are single-purpose — one public method (`execute` or semantically named)
- Modules export only what other modules need to consume
- Shared logic used by two or more modules goes into `shared/`
- Never use `new ServiceClass()` — always inject via constructor
- Avoid circular imports; extract to `shared/` when two modules depend on each other
- `types/` — domain-specific TypeScript types and enums (not DTOs, not entities)
- `constants/` — domain-scoped constants in SCREAMING_SNAKE_CASE; never hardcode magic values inline

## Functions

- Short, single purpose — under 20 instructions
- Early returns to avoid nesting:

```typescript
// Prefer
const domain = await this.domainRepository.findById(id);
if (!domain) throw new NotFoundException(`Domain ${id} not found`);
return this.toResponseDto(domain);

// Over
if (domain) {
  return this.toResponseDto(domain);
}
```

- Use `readonly` for injected dependencies
- Prefer immutable updates over mutation
