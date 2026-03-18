# TypeScript Patterns & Anti-Bloat Rules

> Architectural patterns, anti-bloat commandments, and coding rules for writing lean, powerful TypeScript — tuned for AI-assisted development in 2026.

---

## The 10 Anti-Bloat Commandments

1. **Files under 150 lines** — if a file is longer, split by domain concern
2. **Functions over classes** — unless state + methods genuinely belong together
3. **Interfaces over class hierarchies** — simpler contracts, easier testing
4. **Result types over exceptions** — explicit error flow, no hidden control jumps
5. **Named exports only** — better tree-shaking and refactoring
6. **Feature folders over layer folders** — group by domain (`payments/`), not role (`services/`)
7. **Zod at boundaries only** — validate external data, trust internal types
8. **Pure core, I/O shell** — business logic as pure functions, I/O at the edges
9. **No utility dumping grounds** — no `utils.ts`, `helpers.ts`, or `common.ts`
10. **Reject LLM bloat** — actively remove additive patterns that AI generates

---

## Patterns That Kill Bloat

### Result Type (Railway-Oriented Error Handling)

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

const Ok = <T>(value: T): Result<T, never> => ({ ok: true, value });
const Err = <E>(error: E): Result<never, E> => ({ ok: false, error });

// Usage
function parseConfig(raw: string): Result<Config, ParseError> {
  try {
    return Ok(JSON.parse(raw));
  } catch (e) {
    return Err({ code: 'INVALID_JSON', message: String(e) });
  }
}
```

For heavier use cases: `neverthrow` (~2KB gz) for `ResultAsync` with `map`/`andThen` chaining. Only reach for Effect-TS if you genuinely need the fiber runtime.

### Functional Core, Imperative Shell

```typescript
// CORE — pure, easily tested, no I/O
function calculateDiscount(
  price: number, tier: 'bronze' | 'silver' | 'gold'
): number {
  const rates = { bronze: 0.05, silver: 0.10, gold: 0.15 };
  return price * rates[tier];
}

// SHELL — handles I/O, minimal logic
async function applyDiscountEndpoint(req: Request): Promise<Response> {
  const user = await getUser(req.params.id);
  const discount = calculateDiscount(req.body.price, user.tier);
  await saveOrder({ ...req.body, discount });
  return Response.json({ discount });
}
```

The core is trivially testable (pure in, pure out). The shell is thin and mostly wiring.

### Dependency Injection via Interfaces

```typescript
interface EmailSender {
  send(to: string, subject: string, body: string): Promise<boolean>;
}

// Production
class ResendEmailSender implements EmailSender {
  async send(to: string, subject: string, body: string) {
    return await resend.emails.send({ to, subject, html: body });
  }
}

// Test
class MockEmailSender implements EmailSender {
  sent: Array<{ to: string; subject: string }> = [];
  async send(to: string, subject: string) {
    this.sent.push({ to, subject });
    return true;
  }
}
```

No decorators, no DI framework needed for most projects. Just interfaces + constructor/function params.

### Branded Types for Domain Safety

```typescript
import { z } from 'zod';

const UserId = z.string().uuid().brand('UserId');
const OrderId = z.string().uuid().brand('OrderId');

type UserId = z.infer<typeof UserId>;
type OrderId = z.infer<typeof OrderId>;

// Compiler prevents: getUser(orderId) — type error!
function getUser(id: UserId): Promise<User> { /* ... */ }
```

### Type-Safe Event Maps

```typescript
type EventMap = {
  'user:created': { id: string; email: string };
  'order:completed': { id: string; total: number };
  'payment:failed': { orderId: string; reason: string };
};

type EventHandler<K extends keyof EventMap> = (data: EventMap[K]) => void;
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
├── notifications/
│   ├── types.ts
│   ├── email.ts
│   └── api.ts
└── shared/            # Only truly shared code
    ├── result.ts      # Result type utilities
    ├── config.ts      # App configuration
    └── logger.ts      # Structured logging
```

**Never this:** `models/` + `services/` + `controllers/` + `utils/` + `helpers/`

---

## LLM Bloat Patterns to Reject

AI coding assistants tend to generate these — catch and remove them:

| Pattern | Problem | Fix |
|---------|---------|-----|
| `utils.ts` / `helpers.ts` | Attracts unrelated code | Name files by what they do |
| Try-catch wrapping every function | Hides errors, adds noise | Handle at boundaries only |
| `console.log` everywhere | Not structured, leaks to prod | Use a logger or nothing |
| Runtime checking typed data | Redundant work | Trust internal types |
| Classes with one method | Overcomplicated | Use a function |
| Default exports | Kills tree-shaking | Named exports always |
| JSDoc restating types | Noise | Types are the docs |
| Manager → Service → Handler | Enterprise theater | Flatten to functions |
| Config for constants | Over-engineering | Hardcode what doesn't change |
| `any` type | Defeats TypeScript | Use `unknown` + narrowing |

---

## Validation Strategy

```
External data (API requests, file reads, env vars, user input)
    → Validate with Zod at the boundary
    → Convert to typed domain objects
    → Internal code trusts types completely
    → No re-validation inside the system
```

```typescript
// BOUNDARY — validate
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
}).strict();

type CreateUser = z.infer<typeof CreateUserSchema>;

// INTERNAL — trust types
function processUser(data: CreateUser): User {
  return { ...data, id: crypto.randomUUID(), createdAt: new Date() };
}
```

---

## Strict TypeScript Configuration

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true
  }
}
```

Key flag: `noUncheckedIndexedAccess` makes array/record access return `T | undefined`, catching real null bugs.

---

## Toolchain (2026)

| Tool | Purpose | Why |
|------|---------|-----|
| **pnpm** | Package manager | Faster, disk-efficient, catalogs for monorepos |
| **tsx** | Dev runner | Run TypeScript directly, no build step |
| **Vitest 4** | Testing | 10-20x faster than Jest, browser mode, type testing |
| **Biome 2.3** | Lint + format | 19-100x faster than ESLint+Prettier, single config |
| **Zod v4 Mini** | Validation | 1.88KB gz, tree-shakable, full ecosystem |
| **es-toolkit** | Utilities | 97% smaller than lodash, faster |
| **Knip** | Dead code | Finds unused files, exports, deps |
| **neverthrow** | Error handling | 2KB gz Result types |

---

## Testing Philosophy

- **Unit tests** for pure business logic (fast, many)
- **Integration tests** for module boundaries (medium, some)
- **E2E tests** for critical user flows (slow, few)
- **Type tests** with `expectTypeOf` (free, verify contracts)
- **Property tests** with fast-check (find edge cases automatically)
- **Architecture tests** with ArchUnitTS (enforce module boundaries in CI)
