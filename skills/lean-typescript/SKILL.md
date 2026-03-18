---
name: lean-typescript
description: >
  TypeScript 2026 best practices for writing small, powerful codebases. Anti-bloat
  patterns, schema-first validation, modern toolchain (pnpm, tsx, Vitest 4, Biome),
  and architectural patterns that both humans and AI agents can reason about. Use this
  skill whenever writing TypeScript, reviewing TypeScript code, setting up a new TS
  project, choosing libraries, configuring tsconfig, debloating a codebase, or when
  the user mentions: TypeScript, Zod, Vitest, pnpm, "too much code", "clean up",
  "reduce bundle", "strict mode", "type safety", "modern stack", "2026 best practices",
  or any request involving TypeScript architecture, tooling, or code quality.
---

# Lean TypeScript

Write less code that does more. Every line is a liability — TypeScript's type system
is your primary tool for eliminating runtime bloat.

---

## 10 Golden Rules

1. **Files under 150 lines** — forces modular thinking; split by domain concern
2. **Functions over classes** — unless state + methods genuinely belong together
3. **Interfaces over class hierarchies** — simpler contracts, easier testing
4. **Result types over exceptions** — explicit error flow, no hidden control jumps
5. **Zod at boundaries only** — trust static types internally, validate external data
6. **Named exports only** — better tree-shaking and refactoring
7. **Feature folders** — group by domain (`payments/`, `auth/`), not layer (`services/`, `models/`)
8. **Pure core, I/O shell** — business logic as pure functions, I/O at the edges
9. **pnpm + tsx + Vitest** — the 2026 standard toolchain
10. **Reject LLM bloat** — stay vigilant against additive patterns

---

## Anti-Bloat: Patterns to Reject

LLMs default to these — catch and remove them:

- `utils.ts` or `helpers.ts` — be specific about what the module does
- Try-catch wrapping every async function — handle errors at boundaries
- `console.log` scattered everywhere — use structured logging or nothing
- Runtime type checking what TypeScript already guarantees — trust your types
- Classes with only a constructor and one method — use a function
- Default exports — kills tree-shaking, worse DX for refactoring
- JSDoc that restates the type signature — the types *are* the docs
- Manager → Service → Handler hierarchies — flatten to functions + interfaces
- Config options for things that never change — hardcode them
- `any` type — always find the real type or use `unknown` + narrowing

---

## Core Patterns

### Result Type (railway-oriented)

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

const Ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
const Err = <E>(error: E): Result<never, E> => ({ ok: false, error });
```

For heavier use cases, `neverthrow` (~2KB gz) provides `ResultAsync` with `map`/`andThen` chaining. Only reach for Effect-TS if you need its full fiber runtime — it carries ~5-10KB overhead and requires full buy-in.

### Dependency Injection via Interfaces

```typescript
interface PaymentProvider {
  charge(amount: number, token: string): Promise<string>;
}

// Swap implementations without changing callers
async function processPayment(
  provider: PaymentProvider, amount: number, token: string
): Promise<string> {
  return provider.charge(amount, token);
}
```

### Zod at Boundaries

```typescript
import { z } from 'zod';

// Validate external data at the edge
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
}).strict();

type CreateUser = z.infer<typeof CreateUserSchema>;

// Internal functions trust types — no re-validation
function saveUser(data: CreateUser): Promise<User> {
  return db.users.insert(data);
}
```

### Branded Types for Domain Safety

```typescript
const UserId = z.string().uuid().brand('UserId');
type UserId = z.infer<typeof UserId>;

// Compiler prevents mixing UserId with OrderId
function getUser(id: UserId): Promise<User> { /* ... */ }
```

### Functional Core, Imperative Shell

```typescript
// CORE — pure, easily tested, no I/O
function calculateBudget(clicks: number, cpc: number, mult: number): number {
  return clicks * cpc * mult;
}

// SHELL — handles I/O, minimal logic
async function updateBudgetEndpoint(req: Request): Promise<Response> {
  const metrics = await fetchMetrics(req.params.id);
  const budget = calculateBudget(metrics.clicks, metrics.cpc, 1.2);
  await saveBudget(req.params.id, budget);
  return Response.json({ budget });
}
```

### Type-Safe Event System

```typescript
type EventMap = {
  'user.created': { id: string; email: string };
  'order.completed': { id: string; total: number };
};

type EventHandler<T extends keyof EventMap> = (data: EventMap[T]) => void;
```

---

## Project Structure Template

```
src/
├── payments/          # Feature domain
│   ├── types.ts       # Domain types + Zod schemas
│   ├── stripe.ts      # Provider implementation
│   ├── api.ts         # HTTP routes
│   └── events.ts      # Domain events
├── auth/
│   ├── types.ts
│   ├── oauth.ts
│   └── api.ts
└── shared/            # Only truly shared code
    ├── result.ts      # Result type utilities
    └── config.ts      # App configuration
```

**Not this:** `models/` + `services/` + `controllers/` + `utils/` (layer-based = coupling magnet)

---

## Toolchain & Config

For detailed toolchain setup, library choices, and debloating strategies, read the
reference files in `references/`:

- **`references/toolchain.md`** — pnpm, tsx, Vitest 4, Biome 2.3, strict tsconfig, package.json template
- **`references/patterns.md`** — advanced DI patterns (Awilix, InversifyJS), testing strategies, property-based testing with fast-check, architecture testing
- **`references/debloating.md`** — library replacements (es-toolkit, Zod v4 mini, Temporal API), dead code elimination with Knip, bundle analysis

---

## Quick Decision Guide

| Question | Answer |
|----------|--------|
| Class or function? | Function, unless genuine state + methods |
| Zod or manual validation? | Zod at boundaries, nothing internally |
| Effect-TS or neverthrow? | neverthrow unless you need the full runtime |
| Jest or Vitest? | Vitest 4 (10-20x faster) |
| ESLint+Prettier or Biome? | Biome 2.3 (19-100x faster, single config) |
| npm, yarn, or pnpm? | pnpm (faster, disk-efficient, catalogs) |
| lodash or es-toolkit? | es-toolkit (97% smaller, 2-3x faster) |
| moment/dayjs or Temporal? | Temporal API (native, 0KB) if targeting modern runtime |
| Default or named export? | Named. Always. |
