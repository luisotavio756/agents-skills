---
name: nestjs-module
description: >-
  Scaffold a complete NestJS domain module from scratch. Use when creating a new
  resource, domain, or API route group. Generates entity, repository, use-case
  services (each with co-located DTOs), controller, module, and tests following
  project conventions.
---

# NestJS Module Scaffolding

Adapt ORM decorators to the project (TypeORM, Prisma, Drizzle). Follow project-specific conventions first.

## Folder output

```
{domain}/
├── {domain}.module.ts
├── {domain}.controller.ts
├── {domain}.repository.ts
├── entities/
│   └── {domain}.entity.ts
├── types/
│   └── {domain}.types.ts
├── constants/
│   └── {domain}.constants.ts
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
        ├── find-{domain}.service.spec.ts
        └── dto/
            └── find-{domain}-response.dto.ts
```

## Step-by-step

### 1 — Entity

```typescript
// {domain}/entities/{domain}.entity.ts
@Entity('{domain}s')
export class Domain {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### 2 — Repository

Returns `null` on not found — never throws:

```typescript
// {domain}/{domain}.repository.ts
@Injectable()
export class DomainRepository {
  constructor(
    @InjectRepository(Domain)
    private readonly repo: Repository<Domain>,
  ) {}

  async findById(id: string): Promise<Domain | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findAll(): Promise<Domain[]> {
    return this.repo.find();
  }

  async save(data: Partial<Domain>): Promise<Domain> {
    return this.repo.save(data);
  }

  async delete(id: string): Promise<void> {
    await this.repo.delete(id);
  }
}
```

### 3 — Create service + DTOs

```typescript
// services/create-{domain}/dto/create-{domain}.dto.ts
export class CreateDomainDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  name: string;
}

// services/create-{domain}/dto/create-{domain}-response.dto.ts
export class CreateDomainResponseDto {
  id: string;
  name: string;
  createdAt: Date;
}

// services/create-{domain}/create-{domain}.service.ts
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

### 4 — Find service + DTOs

```typescript
// services/find-{domain}/dto/find-{domain}-response.dto.ts
export class FindDomainResponseDto {
  id: string;
  name: string;
  createdAt: Date;
}

// services/find-{domain}/find-{domain}.service.ts
@Injectable()
export class FindDomainService {
  constructor(private readonly domainRepository: DomainRepository) {}

  async findById(id: string): Promise<FindDomainResponseDto> {
    const domain = await this.domainRepository.findById(id);
    if (!domain) throw new NotFoundException(`Domain ${id} not found`);
    return this.toResponseDto(domain);
  }

  async findAll(): Promise<FindDomainResponseDto[]> {
    const domains = await this.domainRepository.findAll();
    return domains.map(this.toResponseDto);
  }

  private toResponseDto(domain: Domain): FindDomainResponseDto {
    return { id: domain.id, name: domain.name, createdAt: domain.createdAt };
  }
}
```

### 5 — Update service + DTOs

```typescript
// services/update-{domain}/dto/update-{domain}.dto.ts
export class UpdateDomainDto extends PartialType(CreateDomainDto) {}

// services/update-{domain}/dto/update-{domain}-response.dto.ts
export class UpdateDomainResponseDto {
  id: string;
  name: string;
  updatedAt: Date;
}

// services/update-{domain}/update-{domain}.service.ts
@Injectable()
export class UpdateDomainService {
  constructor(private readonly domainRepository: DomainRepository) {}

  async execute(id: string, dto: UpdateDomainDto): Promise<UpdateDomainResponseDto> {
    const existing = await this.domainRepository.findById(id);
    if (!existing) throw new NotFoundException(`Domain ${id} not found`);

    const updated = await this.domainRepository.save({ ...existing, ...dto });
    return this.toResponseDto(updated);
  }

  private toResponseDto(domain: Domain): UpdateDomainResponseDto {
    return { id: domain.id, name: domain.name, updatedAt: domain.updatedAt };
  }
}
```

### 6 — Delete service

```typescript
// services/delete-{domain}/delete-{domain}.service.ts
@Injectable()
export class DeleteDomainService {
  constructor(private readonly domainRepository: DomainRepository) {}

  async execute(id: string): Promise<void> {
    const existing = await this.domainRepository.findById(id);
    if (!existing) throw new NotFoundException(`Domain ${id} not found`);
    await this.domainRepository.delete(id);
  }
}
```

### 7 — Controller

One service injected per route — no logic in the controller:

```typescript
// {domain}/{domain}.controller.ts
@Controller('{domain}s')
export class DomainController {
  constructor(
    private readonly createDomainService: CreateDomainService,
    private readonly findDomainService: FindDomainService,
    private readonly updateDomainService: UpdateDomainService,
    private readonly deleteDomainService: DeleteDomainService,
  ) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() dto: CreateDomainDto): Promise<CreateDomainResponseDto> {
    return this.createDomainService.execute(dto);
  }

  @Get()
  findAll(): Promise<FindDomainResponseDto[]> {
    return this.findDomainService.findAll();
  }

  @Get(':id')
  findById(@Param('id') id: string): Promise<FindDomainResponseDto> {
    return this.findDomainService.findById(id);
  }

  @Patch(':id')
  update(
    @Param('id') id: string,
    @Body() dto: UpdateDomainDto,
  ): Promise<UpdateDomainResponseDto> {
    return this.updateDomainService.execute(id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  delete(@Param('id') id: string): Promise<void> {
    return this.deleteDomainService.execute(id);
  }
}
```

### 8 — Module

```typescript
// {domain}/{domain}.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([Domain])],
  controllers: [DomainController],
  providers: [
    DomainRepository,
    CreateDomainService,
    FindDomainService,
    UpdateDomainService,
    DeleteDomainService,
  ],
  exports: [FindDomainService],
})
export class DomainModule {}
```

Register in `AppModule.imports[]`.

### 9 — Unit test (per service)

Co-locate the spec next to the service it tests:

```typescript
// services/find-{domain}/find-{domain}.service.spec.ts
describe('FindDomainService', () => {
  let service: FindDomainService;
  let repository: jest.Mocked<DomainRepository>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        FindDomainService,
        {
          provide: DomainRepository,
          useValue: { findById: jest.fn(), findAll: jest.fn() },
        },
      ],
    }).compile();

    service = module.get(FindDomainService);
    repository = module.get(DomainRepository);
  });

  afterEach(() => jest.clearAllMocks());

  it('returns the mapped DTO when entity exists', async () => {
    repository.findById.mockResolvedValue({ id: '1', name: 'Test', createdAt: new Date() });
    const result = await service.findById('1');
    expect(result).toMatchObject({ id: '1', name: 'Test' });
  });

  it('throws NotFoundException when entity does not exist', async () => {
    repository.findById.mockResolvedValue(null);
    await expect(service.findById('missing')).rejects.toThrow(NotFoundException);
  });
});
```

## Checklist

```
- [ ] Entity created with correct ORM decorators
- [ ] Repository: findById returns null (not throws) when not found
- [ ] Each use case has its own service folder under services/
- [ ] Input DTOs co-located in services/{action}-{domain}/dto/
- [ ] Output DTOs co-located in services/{action}-{domain}/dto/
- [ ] UpdateDto extends PartialType(CreateDto) — no duplication
- [ ] Service throws NotFoundException when entity is null
- [ ] Service has private toResponseDto() — no raw entity returned
- [ ] Controller injects one service per route — no business logic
- [ ] POST returns 201, DELETE returns 204
- [ ] Module registers all services in providers[]
- [ ] Module registered in AppModule
- [ ] Domain errors added to ErrorsCatalog in src/shared/errors/errors-catalog.ts
- [ ] Controller methods have JSDoc summary (no @ApiOperation)
- [ ] Controller uses SetApi*Response decorators — no raw Swagger decorators inline
- [ ] Services throw from ErrorsCatalog — no inline HttpException
- [ ] Spec co-located next to each non-trivial service
```
