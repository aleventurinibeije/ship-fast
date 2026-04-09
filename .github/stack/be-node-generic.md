# Backend Best Practices — Node.js + TypeScript

<!-- DRAFT — minimal starting point. Fill in your project's specific conventions. -->

---

## Mandatory Conventions

1. **No `any` types** — use proper TypeScript interfaces/types everywhere
2. **Dependency injection** via constructor (NestJS) or explicit parameter passing (plain Express) — no module-level singletons imported directly into functions
3. **Config via environment variables** only — use a typed config service or `zod`-validated schema at startup
4. **DB schema via migration tool** (e.g. Prisma Migrate, Knex migrations) — never `sync: true` in production
5. **Error handling:** typed error classes, never `throw new Error("string")` for business errors

---

## TypeScript Patterns

```typescript
// Config validation at startup (zod example)
import { z } from 'zod';
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
});
export const config = envSchema.parse(process.env);

// Service with constructor injection
export class MyService {
  constructor(
    private readonly repo: MyRepository,
    private readonly config: AppConfig,
  ) {}
}
```

---

## Testing Conventions

### Test Types

| Type | Framework | When |
|------|-----------|------|
| Unit | Jest | Isolated functions/classes |
| Integration | Jest + Supertest | HTTP layer, DB interactions |
| E2E | Jest + Supertest | Full request lifecycle |

### Coverage Targets

- **Line coverage:** ≥ 80%
- **Branch coverage:** ≥ 70%

### Unit Test Structure

```typescript
describe('MyService', () => {
  let service: MyService;
  let mockRepo: jest.Mocked<MyRepository>;

  beforeEach(() => {
    mockRepo = { findById: jest.fn(), save: jest.fn() } as any;
    service = new MyService(mockRepo, testConfig);
  });

  describe('processItem', () => {
    it('returns result when item exists', async () => {
      // Arrange
      mockRepo.findById.mockResolvedValue({ id: 1, name: 'test' });

      // Act
      const result = await service.processItem(1);

      // Assert
      expect(result).toMatchObject({ id: 1 });
      expect(mockRepo.findById).toHaveBeenCalledWith(1);
    });

    it('throws NotFoundError when item does not exist', async () => {
      // Arrange
      mockRepo.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(service.processItem(99)).rejects.toThrow(NotFoundError);
    });
  });
});
```

### Test Naming Convention

`describe('ClassName') > describe('methodName') > it('returns X when Y')`

---

## Logging Convention

```typescript
// Business events → info
logger.info({ submissionId }, 'Processing started');

// Failures → error with err field
logger.error({ err, submissionId }, 'Processing failed');

// NEVER log sensitive data (tokens, passwords, PII)
```
