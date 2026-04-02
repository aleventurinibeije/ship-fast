# Backend Best Practices — Node.js + TypeScript + Platformatic DB

---

## Stack Overview

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js >= 22.19.0 |
| Framework | **Platformatic DB** v3.x (Fastify v5 underneath) |
| Orchestrator | **WattPM** — gateway → platformatic-db two-service app |
| Language | TypeScript 5.5 — ESM, strict mode, `moduleResolution: nodenext` |
| Database | PostgreSQL via `pg` — no ORM, Platformatic CRUD auto-generation |
| Migrations | Raw SQL files (`migrations/NNN.do.sql` / `NNN.undo.sql`) |
| Auth | **Logto** (OIDC/JWT) + Platformatic JWT middleware |
| Schema validation | **TypeBox** (`@sinclair/typebox`) |
| Package manager | **pnpm** workspace monorepo |
| Formatter | **Prettier** (no ESLint) |

---

## Mandatory Conventions

1. **No `any` types** — strict TypeScript everywhere; use `interface` for shapes, `type` for unions and aliases.
2. **No ORM** — access entities via `app.platformatic.entities.<entity>.find/save/insert/delete(...)`. Never write raw SQL in route handlers.
3. **TypeBox schemas for all route I/O** — define `const FooSchema = Type.Object(...)` and derive `type Foo = Static<typeof FooSchema>`. Co-locate schema and type in the same route file.
4. **Migrations only** — database changes via numbered SQL migration files. Never alter the DB schema directly or rely on auto-sync.
5. **Env vars via `watt.json` interpolation** — use `{PLACEHOLDER_NAME}` syntax. Read at runtime from `process.env` in plugins. No `dotenv`, no Zod validation at startup.
6. **Typed error classes** — use `LogtoHttpError` (or a project-specific subclass) for business errors. Never `throw new Error("string")` for recoverable failures.
7. **Error response shape is always `{ error: string, message: string }`** — all route error paths must return this structure.

---

## TypeScript Patterns

```typescript
// Entity access — always use Platformatic entity API
const residents = await app.platformatic.entities.resident.find({
  where: { buildingId: { eq: currentUser.buildingId } },
  skipAuth: true,  // for admin/background ops only
})

// TypeBox schema + derived type (co-located in route file)
import { Type, Static } from '@sinclair/typebox'

const CreateWorkerBodySchema = Type.Object({
  name: Type.String(),
  buildingId: Type.Number(),
})
type CreateWorkerBody = Static<typeof CreateWorkerBodySchema>

// Route with Fastify generics
fastify.post<{ Body: CreateWorkerBody }>('/workers', {
  schema: { body: CreateWorkerBodySchema },
}, async (req, reply) => { ... })

// Plugin signature — all plugins are async default-exported functions
import type { FastifyInstance } from 'fastify'
export default async function myPlugin(app: FastifyInstance) {
  app.decorate('myHelper', ...)
}

// Fastify module augmentation for decorators
declare module 'fastify' {
  interface FastifyInstance {
    requireScope: (scope: string) => preHandlerHookHandler
    buildingScope: (req: FastifyRequest) => { eq: number } | null
  }
  interface FastifyRequest {
    currentUser: CurrentUser
  }
}
```

---

## Auth Patterns

All requests to `/api/*` go through Platformatic's JWT middleware. Custom routes under `/admin/*` call `req.jwtVerify()` explicitly in a `preValidation` hook.

### `currentUser` decorator

After `preValidation`, `req.currentUser` is always available:

```typescript
interface CurrentUser {
  id: number
  accessTier: AccessTier        // 'global' | 'tenant'
  buildingId: number | null
  companyId: number | null
  scopeMap: ScopeMap
}
```

### Access tier logic

| Condition | `accessTier` | Effect |
|-----------|-------------|--------|
| JWT has `user-svc:admin` scope | `"global"` | Full access, no row-level filters |
| Any other JWT | `"tenant"` | Row-level filters via `buildingId` checks |

### Scope guard

```typescript
// Protect a route with a Logto scope
fastify.post('/workers', {
  preHandler: [fastify.requireScope('user-svc:workers:write')],
}, handler)
```

### Row-level filtering

```typescript
// In a tenant-scoped route
const where = fastify.buildingScope(req) // { eq: buildingId } or null
const items = await app.platformatic.entities.worker.find({ where: { buildingId: where } })
```

### Scope string format

Scopes follow the pattern `<resource>:<entity>:<action>`, e.g. `user-svc:workers:read`. The wildcard `*` and the `admin` super-scope shortcut are supported by `scope-utils.ts`.

---

## Error Handling

```typescript
// Typed error class
class LogtoHttpError extends Error {
  constructor(
    public readonly statusCode: number,
    message: string,
    public readonly code?: string,
  ) { super(message) }
}

// Error helper — use in every catch block
function logtoError(err: unknown): { code: number; payload: object } {
  if (err instanceof LogtoHttpError) {
    return {
      code: err.statusCode,
      payload: { error: err.code ?? `LogtoError${err.statusCode}`, message: err.message },
    }
  }
  return { code: 503, payload: { error: 'LogtoUnavailable', message: 'Logto non raggiungibile' } }
}

// In route handler
try {
  await doLogtoOperation()
} catch (err) {
  const { code, payload } = logtoError(err)
  return reply.code(code).send(payload)
}

// Auth/forbidden errors — reply directly, no throw
return reply.code(403).send({ error: 'Forbidden', message: 'Insufficient permissions' })
```

---

## Database — Migrations

Migration files live in `web/platformatic-db/migrations/`. Named `NNN.do.sql` / `NNN.undo.sql`.

```sql
-- Naming: snake_case tables and columns
-- Prefer ENUMs defined as separate lookup tables (not PostgreSQL ENUM type)
-- Use PostgreSQL functions for complex atomic operations
-- Partial unique indexes preferred over table-level unique constraints where appropriate

-- Example: 001.do.sql
CREATE TABLE buildings (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  company_id INTEGER REFERENCES companies(id) ON DELETE CASCADE
);
```

Migration auto-apply is controlled by `PLT_PLATFORMATIC_DB_APPLY_MIGRATIONS`. Always `true` in tests, `false` in production (apply separately).

---

## Testing

**Framework**: Node.js built-in `node:test` + `node:assert/strict`. No Jest, no Vitest.

### Test types

| Type | Setup | When |
|------|-------|------|
| Unit (plugin/util) | Bare `Fastify()` instance + `app.inject()` | Isolated plugin/util logic |
| Integration (route) | `getServer(t)` from `test/helper.ts` — full Platformatic app + real DB | Route behaviour end-to-end |
| Config/structural | File-parsing assertions (no server) | Validate `watt.json` RBAC rules vs. entity types |

### Running tests

```bash
# Development (TypeScript directly)
node --test test/**/*.test.ts

# CI / after build
npm run build && node --test dist/test/**/*.test.js
```

### Unit test structure

```typescript
import { test } from 'node:test'
import assert from 'node:assert/strict'
import Fastify from 'fastify'
import myPlugin from '../../plugins/my-plugin.ts'

test('myPlugin — does X when Y', async (t) => {
  const app = Fastify({ logger: { level: 'warn' } })
  await app.register(myPlugin)
  await app.ready()
  t.after(() => app.close())

  const res = await app.inject({ method: 'GET', url: '/path' })
  assert.equal(res.statusCode, 200)
  assert.deepEqual(JSON.parse(res.body), { expected: true })
})
```

### Integration test structure

```typescript
import { test } from 'node:test'
import assert from 'node:assert/strict'
import { getServer } from '../helper.ts'

test('GET /api/workers — returns tenant-scoped workers', async (t) => {
  const { app } = await getServer(t)

  const res = await app.inject({
    method: 'GET',
    url: '/api/workers',
    headers: { 'X-PLATFORMATIC-ACCESS-TIER': 'tenant', 'X-PLATFORMATIC-BUILDING-ID': '1' },
  })

  assert.equal(res.statusCode, 200)
})
```

### Mocking fetch (for Logto HTTP calls)

```typescript
// Replace globalThis.fetch with ordered handler array; restore in t.after()
const handlers: Array<() => Promise<Response>> = [
  async () => new Response(JSON.stringify({ access_token: 'tok' }), { status: 200 }),
]
const originalFetch = globalThis.fetch
globalThis.fetch = async () => handlers.shift()!()
t.after(() => { globalThis.fetch = originalFetch })
```

### Stubbing Fastify decorators (for route unit tests)

```typescript
// No mocking library — decorate a bare Fastify instance manually
const app = Fastify({ logger: { level: 'warn' } })
app.decorate('logto', { createUser: async () => ({ id: 'usr_1' }) })
app.decorate('requireScope', (_scope: string) => async () => {})
await app.register(myRoute)
```

---

## Logging

Pino (Fastify default). Log level controlled by `PLT_SERVER_LOGGER_LEVEL`.

```typescript
// Business events → info
req.log.info({ userId: req.currentUser.id }, 'Worker created')

// Failures → error with err field
req.log.error({ err, userId: req.currentUser.id }, 'Logto user creation failed')

// In tests — always set level to 'warn' to suppress noise
const app = Fastify({ logger: { level: 'warn' } })

// NEVER log: tokens, passwords, full JWT payloads, PII
```

---

## Project Structure — Quick Reference

```
web/platformatic-db/
├── migrations/          ← NNN.do.sql / NNN.undo.sql  (raw SQL, sequential)
├── plugins/             ← Fastify plugins: decorate app, register hooks
│   ├── logto.ts         ← LogtoService (M2M client, user/role/scope CRUD)
│   ├── multitenancy.ts  ← JWT auth, currentUser, requireScope, buildingScope
│   ├── role-utils.ts    ← Role name constants & validation
│   └── scope-utils.ts   ← Scope string → Map parser
├── routes/              ← Route handlers
│   ├── root.ts
│   └── admin/           ← /admin/* routes (Logto-proxied operations)
├── schemas/             ← Shared TypeBox schemas (reused across routes)
├── test/
│   ├── helper.ts        ← getServer() — creates isolated PostgreSQL DB per test
│   ├── config/          ← Structural/config assertions
│   ├── plugins/         ← Plugin unit tests (bare Fastify)
│   └── routes/          ← Route tests (unit with stubs OR integration via getServer)
├── types/               ← Auto-generated entity types (do not edit manually)
└── watt.json            ← Platformatic config: DB, migrations, auth rules, plugins, routes
```

### Key env vars

| Variable | Purpose |
|----------|---------|
| `PLT_PLATFORMATIC_DB_DATABASE_URL` | PostgreSQL connection string |
| `PLT_PLATFORMATIC_DB_APPLY_MIGRATIONS` | `true` in tests, `false` in prod |
| `PLT_ADMIN_SECRET` | Platformatic admin bypass secret |
| `PLT_JWKS_ALLOWED_DOMAIN` | Logto OIDC domain for JWKS discovery |
| `PLT_LOGTO_ENDPOINT` | Logto API base URL |
| `PLT_LOGTO_M2M_APP_ID` | M2M client ID |
| `PLT_LOGTO_M2M_APP_SECRET` | M2M client secret |
| `PLT_LOGTO_API_RESOURCE` | Logto API resource identifier |
| `PLT_SERVER_LOGGER_LEVEL` | Pino log level (`info`, `warn`, `error`) |
