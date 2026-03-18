# Advanced Patterns Reference

## Dependency Injection

### Awilix (Decorator-Free DI)

Zero decorators, zero reflect-metadata, full Vite/esbuild compatibility. Scoped containers for isolation.

```typescript
import { createContainer, asClass, asValue } from 'awilix';

const container = createContainer();
container.register({
  mediaStore: asClass(MediaStore).singleton(),
  paymentService: asClass(PaymentService).scoped(),
});

// Per-module isolation
const scope = container.createScope();
scope.register({ config: asValue(moduleConfig) });
```

Why Awilix over InversifyJS: TC39 standard decorators (TS 5.0+) broke `emitDecoratorMetadata`. Decorator-based DI is architectural debt in 2026.

### When to skip DI containers

For smaller codebases (<20 files), a composition root pattern works:

```typescript
// src/composition-root.ts
export function createApp(config: AppConfig) {
  const db = createDatabase(config.dbUrl);
  const stripe = createStripeClient(config.stripeKey);
  const payments = createPaymentService(stripe, db);
  return { payments, db };
}
```

---

## Testing Strategies

### Property-Based Testing (fast-check)

Test invariants, not examples. fast-check v4.3 with shrinking and model-based testing.

```typescript
import fc from 'fast-check';

it('serialization is lossless', () => {
  fc.assert(fc.property(
    fc.record({ name: fc.string({ minLength: 1 }), version: fc.string() }),
    (config) => expect(JSON.parse(JSON.stringify(config))).toEqual(config)
  ));
});

it('budget is never negative', () => {
  fc.assert(fc.property(
    fc.integer({ min: 0, max: 100000 }),
    fc.float({ min: 0, max: 10 }),
    (clicks, cpc) => calculateBudget(clicks, cpc) >= 0
  ));
});
```

### Type-Level Testing (built into Vitest)

```typescript
import { expectTypeOf } from 'vitest';

test('API types are correct', () => {
  expectTypeOf<CreateUser>().toHaveProperty('email');
  expectTypeOf(createUser).parameter(0).not.toBeAny();
});
```

### Mutation Testing (StrykerJS)

Finds tests that pass even when code is mutated — weak test detection.

```json
{
  "testRunner": "vitest",
  "mutate": ["src/**/*.ts"],
  "checkers": ["typescript"],
  "incremental": true
}
```

Target: 80%+ mutation score for core logic, 60%+ for UI code.

### Architecture Testing (ArchUnitTS)

Enforce module boundaries as tests:

```typescript
import { filesOfProject } from 'archunit-ts';

test('payments never imports from auth', async () => {
  const rule = filesOfProject()
    .inFolder('payments')
    .shouldNot().dependOnFiles()
    .inFolder('auth');
  await expect(rule).toPassAsync();
});
```

---

## Plugin Architecture (for larger apps)

### Micro-Kernel Pattern

```typescript
interface Plugin {
  readonly id: string;
  readonly manifest: PluginManifest;
  onInit(ctx: PluginContext): Promise<void>;
  onActivate(): Promise<void>;
  onDeactivate(): Promise<void>;
}

interface PluginManifest {
  id: string;
  version: string;
  activationEvents: string[];
  contributes: {
    commands?: CommandContribution[];
    panels?: PanelContribution[];
  };
}
```

### Event Bus: tseep

381 bytes gzipped, 89M ops/sec. Full NodeJS.EventEmitter interface with generics.

25x faster than mitt by avoiding spread operator — passes args through CPU registers.

### Zustand Slices for Plugin State

```typescript
const createPaymentSlice: StateCreator<PaymentSlice> = (set) => ({
  transactions: [],
  addTransaction: (t) => set(s => ({ transactions: [...s.transactions, t] })),
});
```

Guidelines: multiple small stores > one monolith. Always use selectors. Wrap store access in custom hooks.

---

## Effect-TS (when you actually need it)

Only use if you need: typed dependency injection, fiber-based concurrency, built-in OpenTelemetry, or Rust-style error channels. Carries ~5-10KB overhead and requires full buy-in.

```typescript
import { Effect, Layer, Context } from 'effect';

class PaymentProvider extends Context.Tag('PaymentProvider')<
  PaymentProvider,
  { readonly charge: (amount: number) => Effect.Effect<string, PaymentError> }
>() {}

const processPayment = (amount: number) =>
  Effect.gen(function* () {
    const provider = yield* PaymentProvider;
    return yield* provider.charge(amount);
  });
```

For most projects, `neverthrow` (~2KB) is sufficient for Result types.
