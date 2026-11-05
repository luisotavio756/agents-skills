# Testing

## Unit tests (services)

Test services in isolation — mock all injected dependencies:

```typescript
describe('UserService', () => {
  let service: UserService;
  let repository: jest.Mocked<UserRepository>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: UserRepository,
          useValue: {
            findById: jest.fn(),
            findByEmail: jest.fn(),
            save: jest.fn(),
            delete: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(UserService);
    repository = module.get(UserRepository);
  });

  afterEach(() => jest.clearAllMocks());

  it('throws NotFoundException when user does not exist', async () => {
    repository.findById.mockResolvedValue(null);
    await expect(service.findById('missing-id')).rejects.toThrow(NotFoundException);
  });

  it('returns the mapped response DTO when user exists', async () => {
    repository.findById.mockResolvedValue(createUserEntity({ id: '1', name: 'Ada' }));
    const result = await service.findById('1');
    expect(result).toMatchObject({ id: '1', name: 'Ada' });
  });
});
```

## E2E tests (controllers)

Test the full HTTP stack with `supertest`:

```typescript
describe('UserController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
    app.useGlobalFilters(new GlobalExceptionFilter());
    await app.init();
  });

  afterAll(() => app.close());

  it('GET /users/:id → 200 with user payload', async () => {
    const { body } = await request(app.getHttpServer())
      .get('/users/existing-id')
      .expect(200);

    expect(body).toMatchObject({ id: 'existing-id', name: expect.any(String) });
  });

  it('GET /users/:id → 404 when user not found', async () => {
    await request(app.getHttpServer())
      .get('/users/nonexistent')
      .expect(404);
  });
});
```

## Conventions

- **AAA**: Arrange → Act → Assert structure in every test
- **Naming**: describe what happens, not what method is called
  - `it('throws NotFoundException when user does not exist')` ✓
  - `it('tests findById')` ✗
- **Fixtures**: use factory functions for test data

```typescript
function createUserEntity(overrides: Partial<User> = {}): User {
  return {
    id: 'user-1',
    name: 'Test User',
    email: 'test@example.com',
    role: UserRole.Member,
    createdAt: new Date(),
    ...overrides,
  };
}
```

## What to test

| Layer | Focus |
|-------|-------|
| Service | Business rules, error throwing, edge cases, orchestration |
| Controller | Route binding, guard application, response shape (via e2e) |
| Repository | Query correctness (integration test with test DB) |
| Guard | `canActivate` logic with mocked `ExecutionContext` |
| Pipe | Transform and validation logic with direct instantiation |

## Avoid

- Testing NestJS framework internals (guards applied by NestJS itself)
- Asserting on implementation details — assert outcomes
- Skipping `afterAll(app.close())` in e2e — leaks open handles
- Sharing mutable state between tests without `beforeEach` reset
