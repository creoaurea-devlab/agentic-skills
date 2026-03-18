---
name: spec-driven
description: >
  Contract-first, schema-driven development methodology. Four pillars: Spec (human intent),
  Schema (machine validation), Registry (discovery), and Isolated Modules (testable units).
  Use this skill when designing new modules, APIs, or features from scratch. Also trigger
  when the user mentions: "spec first", "contract first", "schema driven", "API design",
  "new module", "new feature architecture", "how should I structure this", or when building
  something that needs clear boundaries between components. Especially useful for AI-assisted
  development because schemas become the shared language between human intent and machine
  implementation.
---

# Spec-Driven Development

Define the contract before writing the implementation. This methodology ensures that
both humans and AI agents work from the same source of truth — the schema.

---

## The 4 Pillars

### 1. SPEC — Human-Readable Intent

A plain-language document describing *what* the module does, *why* it exists, and
*what* its boundaries are. Written in markdown.

```markdown
# Payment Processing Module

## Purpose
Process credit card payments through Stripe, with fallback to manual invoice.

## Boundaries
- Accepts: validated payment requests (amount, currency, token)
- Returns: payment result (success with ID, or failure with reason)
- Does NOT: handle user authentication, manage subscriptions, send emails

## Dependencies
- Stripe API client (injected)
- Database connection (injected)
```

The spec answers three questions:
1. What does this module accept and return?
2. What is explicitly *outside* its responsibility?
3. What does it depend on (injected, not imported)?

### 2. SCHEMA — Machine-Readable Validation

The spec, translated into runtime-checkable contracts. In TypeScript, use Zod. In Python, use Pydantic.

```typescript
import { z } from 'zod';

export const PaymentRequestSchema = z.object({
  amount: z.number().positive(),
  currency: z.enum(['USD', 'EUR', 'GBP']),
  token: z.string().min(1),
});

export const PaymentResultSchema = z.discriminatedUnion('status', [
  z.object({ status: z.literal('success'), paymentId: z.string() }),
  z.object({ status: z.literal('failure'), reason: z.string() }),
]);

export type PaymentRequest = z.infer<typeof PaymentRequestSchema>;
export type PaymentResult = z.infer<typeof PaymentResultSchema>;
```

The schema is the **single source of truth** for types. Never hand-write a TypeScript
interface that duplicates what Zod already defines — use `z.infer<>`.

### 3. REGISTRY — Discovery & Wiring

A manifest that tells the system (and AI agents) what modules exist, what they expose,
and how to find them.

```json
{
  "modules": [
    {
      "id": "payments",
      "entry": "src/payments/index.ts",
      "schemas": "src/payments/types.ts",
      "depends_on": ["database", "stripe-client"],
      "exposes": ["processPayment", "refundPayment"]
    }
  ]
}
```

The registry enables:
- **AI agents** to discover what already exists before creating new code
- **Humans** to navigate the codebase without reading every file
- **Tools** to validate dependencies and detect circular imports

### 4. ISOLATED MODULE — Single-Responsibility Unit

The implementation. Every module:

- **Receives dependencies via injection** (constructor args or function params)
- **Imports only from shared types** — never reaches into another module's internals
- **Has one public entry point** (`index.ts` with named exports)
- **Is testable in isolation** — mock the injected deps, test the logic

```typescript
// src/payments/process.ts
export async function processPayment(
  provider: PaymentProvider,
  db: Database,
  request: PaymentRequest
): Promise<PaymentResult> {
  const result = await provider.charge(request.amount, request.currency, request.token);
  if (result.ok) {
    await db.payments.insert({ id: result.value, ...request });
    return { status: 'success', paymentId: result.value };
  }
  return { status: 'failure', reason: result.error.message };
}
```

---

## The Flow

```
Human writes SPEC (what & why)
    ↓
Developer/AI creates SCHEMA (Zod/Pydantic types)
    ↓
Register module in REGISTRY (discovery manifest)
    ↓
Implement ISOLATED MODULE against the schema
    ↓
Test against the schema contract
    ↓
If schema drifts from intent → update SPEC → repeat
```

---

## Why This Works for AI-Assisted Development

| Without spec-driven | With spec-driven |
|---------------------|-----------------|
| AI guesses module boundaries | AI reads the spec and knows exactly what to build |
| Duplicate modules appear | Registry prevents creating what already exists |
| Types drift from validation | Schema is the single source for both |
| Tests mock random internals | Tests validate against the contract |
| Refactoring breaks unknown consumers | Registry tracks dependencies explicitly |

---

## Per-Module Artifact Checklist

For each new module, create:

- [ ] `spec.md` — plain-language description of purpose, boundaries, dependencies
- [ ] `types.ts` — Zod schemas + inferred types (or `schemas.py` for Pydantic)
- [ ] `index.ts` — public API (named exports only)
- [ ] Registry entry in project manifest
- [ ] At least one test per schema-defined behavior

---

## Anti-Patterns

- **Schema after implementation** — defeats the purpose; the schema *is* the design
- **God schemas** that validate everything in one file — one schema per boundary
- **Registry that nobody updates** — enforce in CI or don't bother
- **Modules that import from sibling internals** — always go through the public API
- **Specs that describe *how*  instead of *what*** — implementation details belong in code, not specs
