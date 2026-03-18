# The TypeScript Stack for 2025–2026

> Concrete library choices, code patterns, and configuration — from a 381-byte event bus to mutation testing — with version numbers and bundle sizes current through early 2026.

These patterns apply broadly to any complex TypeScript application, whether it's an Electron desktop app, a React web app, or a Node.js API server.

---

## Part 1 — Plugin Architecture

### The Micro-Kernel Pattern

A plugin interface with lifecycle hooks and a manifest-driven contribution point system. Proven at scale by VS Code and Obsidian.

```typescript
interface Plugin {
  readonly id: string;
  readonly manifest: PluginManifest;
  onInit(ctx: PluginContext): Promise<void>;
  onActivate(): Promise<void>;
  onDeactivate(): Promise<void>;
  onDestroy(): Promise<void>;
}

interface PluginManifest {
  id: string;
  version: string;
  activationEvents: string[];
  contributes: {
    nodeTypes?: NodeTypeContribution[];
    panels?: PanelContribution[];
    commands?: CommandContribution[];
    services?: ServiceContribution[];
  };
}
```

The `PluginContext` object (injected during `onInit`) is the plugin's only window into the host. It exposes scoped services, a typed event bus, and registration methods that return disposables. Every subscription returns an `IDisposable`, collected in a `DisposableStore` that auto-cleans on plugin deactivation — eliminating memory leak bugs.

### Event Bus: tseep

| Library | Bundle (gzipped) | Ops/sec | TypeScript Quality |
|---------|-----------------|---------|-------------------|
| **tseep v1.3.x** | **381 bytes** | **89,030,882** | Full generic typing, NodeJS.EventEmitter compatible |
| mitt v3.0.1 | ~200 bytes | 3,587,734 | Good generic type map |
| emittery v1.2.0 | ~1.4 KB | Async-native | Strong typing, Promise-based |
| eventemitter3 v5.0.4 | ~1 KB | 5,698,255 | Basic typing |

tseep achieves ~25x the throughput of mitt by avoiding the spread operator and passing arguments directly through CPU registers (up to 5 args optimized).

### Dependency Injection: Awilix v12

The TC39 standard decorators shipping in TypeScript 5.0+ broke `emitDecoratorMetadata` for runtime reflection. **Decorator-based DI containers (inversify, older tsyringe) are architectural debt in 2026.**

Awilix: zero decorators, zero reflect-metadata, full Vite/esbuild compatibility, first-class scoped containers.

```typescript
import { createContainer, asClass, asValue } from 'awilix';

const container = createContainer();
container.register({
  mediaStore: asClass(MediaStore).singleton(),
  generationService: asClass(GenerationService).scoped(),
});

// Per-plugin isolation
const pluginScope = container.createScope();
pluginScope.register({ pluginConfig: asValue(manifest) });
```

Alternative: `@wroud/di` (2025 newcomer, .NET-inspired ServiceCollection). For smaller codebases, a plain composition root (factory function wiring all services) works fine.

### Command Pattern + CQRS

For undo/redo without state snapshots:

```typescript
interface ICommand {
  execute(): void;
  undo(): void;
}

class MoveNodeCommand implements ICommand {
  constructor(
    private nodeId: string,
    private store: CanvasStore,
    private from: Position,
    private to: Position
  ) {}
  execute() { this.store.setNodePosition(this.nodeId, this.to); }
  undo() { this.store.setNodePosition(this.nodeId, this.from); }
}
```

Commands flow through a Mediator that decouples senders from handlers — lightweight CQRS for the frontend.

### Feature-Sliced Design

```
src/
├── app/              # Initialization, providers, global config
├── widgets/          # Toolbar, browser, inspector
├── features/         # add-node, generate-video, connect-nodes
├── entities/         # node, connection, media-asset, project
├── shared/           # UI kit, lib, config
└── plugins/
    ├── core/         # PluginRegistry, PluginContext, PluginLoader
    └── builtin/      # Each feature as a plugin
        ├── video-filter-node/
        │   ├── manifest.json
        │   ├── VideoFilterPlugin.ts
        │   ├── VideoFilterNode.tsx
        │   ├── commands/
        │   ├── model/
        │   └── index.ts    # Public API only
        └── ai-generation-panel/
```

**Import rule:** higher layers depend on lower layers, never the reverse.

**Barrel exports:** use only at plugin boundaries. Internally, use explicit deep imports. Mark all barrels as `sideEffects: false`.

### Zustand Slices for Plugin State

```typescript
const createVideoFilterSlice: StateCreator<VideoFilterSlice> = (set) => ({
  filters: [],
  addFilter: (f) => set(s => ({ filters: [...s.filters, f] })),
});
```

Guidelines: multiple small stores > one monolith. Always use selectors. Wrap all store access in custom hooks. Use `persist` with `partialize` to save only selected fields.

---

## Part 2 — Code Debloating

### Zod v4 Mini vs Valibot

| Library | Typical schema (gz) | Full library (gz) | Tree-shakable |
|---------|-------------------|-------------------|--------------|
| Zod v3 | ~15–17 KB | ~12.8 KB | Poor |
| **Zod v4 Mini** | **~1.88 KB (Rolldown)** | ~7–8 KB | **Excellent** |
| Valibot | ~1.37 KB | ~5.3 KB | Excellent |

Zod v4: ground-up rewrite, 14x faster string parsing, 7x faster arrays. `zod/v4-mini` is functional and tree-shakable. Valibot wins on raw size by ~73%, but Zod's ecosystem (`.toJSONSchema()`, tRPC, React Hook Form resolvers) makes it the pragmatic choice.

### es-toolkit Replaces lodash (97% Smaller)

| Function | es-toolkit | lodash-es | Reduction |
|----------|-----------|-----------|-----------|
| sample | 88 bytes | 2,000 bytes | 96% |
| omit | ~200 bytes | ~5 KB | 96% |
| difference | ~100 bytes | ~3 KB | 97% |

Used by Storybook, MUI, Recharts. 2–3x faster, up to 11x for specific functions. Migration: `import { pick } from 'lodash-es'` → `import { pick } from 'es-toolkit'`.

For functional composition, pair with **Remeda** (TypeScript-first, lazy evaluation in pipes, 9.08 KB gz).

### Temporal API — Zero Bundle Cost

Landed in Chrome 144 (Jan 2026) and Firefox 139. For modern runtimes:

```typescript
const today = Temporal.Now.plainDateISO();
const deadline = today.add({ days: 14 });
const duration = Temporal.Duration.from({ hours: 2, minutes: 30 });
```

Eliminates: moment.js (~18 KB gz), Luxon (~23 KB gz), dayjs (~2 KB gz).

### neverthrow Over Effect-TS

Effect-TS is powerful but the fiber runtime adds ~5–10 KB and requires full buy-in. `neverthrow` (~2 KB gz) provides `Result<T, E>` and `ResultAsync` with `map`, `mapErr`, `andThen`. `@praha/byethrow` (<1 KB gz) is even lighter.

### Knip: Dead Code Elimination

Gold standard — supersedes ts-prune, unimported, depcheck. Finds unused files, exports, dependencies, and enum members. At Vercel, Knip helped delete ~300,000 lines.

```bash
pnpm add -D knip && npx knip
```

80+ framework plugins. Run `--production` mode for production entry points. Run in CI as a required check.

### TypeScript 5.5–5.8 Features

- **Inferred type predicates (5.5)** — `.filter(x => x !== null)` returns `T[]` not `(T | null)[]`
- **`satisfies` operator** — preserves literal types while enforcing structure
- **`using` keyword (5.2)** — automatic resource cleanup via `Symbol.dispose`
- **`erasableSyntaxOnly` (5.8)** — direct TS execution in Node.js 23.6+
- **TypeScript 7 (Go rewrite)** — 10–15x faster type-checking (coming)

### Estimated Savings

| Swap | Savings (gzipped) |
|------|--------------------|
| lodash → es-toolkit | ~20–24 KB |
| Zod v3 → Zod v4 Mini | ~10–15 KB |
| moment.js → Temporal | ~18 KB |
| Effect-TS → neverthrow | ~5–10 KB |
| Knip dead code removal | 10–30% additional |
| **Total typical** | **~50–70 KB + dead code** |

---

## Part 3 — Testing & Quality

### Vitest 4 (Oct 2025)

Browser mode stable, built-in visual regression testing, Playwright trace integration. `projects` mode replaces `vitest.workspace`.

```typescript
export default defineConfig({
  test: {
    projects: ['packages/*', 'apps/*'],
    coverage: {
      provider: 'v8',
      thresholds: { statements: 80, branches: 75, functions: 80, lines: 80 },
    },
  },
});
```

### Biome 2.3 (Unified Linter-Formatter)

423+ rules, 10–100x faster than ESLint + Prettier. Single `biome.json` replaces 3–4 config files and 127+ npm packages.

Benchmark: 63ms vs ESLint's 1,200ms on 312 files.

Complement with **Oxlint 1.0** (50–100x faster than ESLint, 695+ rules) as a fast pre-commit pass.

### Maximum-Strict tsconfig

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
    "erasableSyntaxOnly": true,
    "moduleResolution": "bundler",
    "module": "ESNext",
    "target": "ES2022",
    "noEmit": true,
    "incremental": true,
    "isolatedModules": true
  }
}
```

### Five-Layer Testing Pyramid

1. **Architecture tests** (ArchUnitTS) — plugin boundaries, no circular deps
2. **Unit tests** (Vitest) — functions, registry, lifecycle hooks, type-level tests
3. **Component tests** (Vitest Browser Mode + Playwright) — rendering in real browser
4. **Integration tests** (Vitest) — plugin loader with real plugins, event bus, store
5. **E2E tests** (Playwright) — full app flows

### The Complete Stack

| Layer | Tool | Size/Speed |
|-------|------|-----------|
| Event bus | tseep v1.3 | 381 bytes, 89M ops/sec |
| DI container | Awilix v12 | No decorators, scoped containers |
| Schema validation | Zod v4 Mini | ~1.88 KB gz |
| Utilities | es-toolkit + Remeda | 97% smaller than lodash |
| Error handling | neverthrow | ~2 KB gz |
| State management | Zustand v5 (slices) | Used by React Flow itself |
| Date/time | Temporal API (native) | 0 KB |
| Linting + formatting | Biome 2.3 + Oxlint 1.x | 19–100x faster than ESLint |
| Dead code | Knip | 80+ framework plugins |
| Unit testing | Vitest 4 | Projects mode, browser mode |
| E2E testing | Playwright | Trace integration |
| Type testing | expect-type (built-in) | Zero extra deps |
| Property testing | fast-check v4.3 | Model-based testing |
| Mutation testing | StrykerJS 7 + vitest-runner | Incremental mode |
| Architecture testing | ArchUnitTS | Enforceable boundaries |
| Build | Vite / electron-vite v5 | HMR for all processes |
| Monorepo | pnpm catalogs + Turborepo | Affected-only CI |
| TypeScript | 5.8+ (maximum strict) | Go rewrite coming (10–15x faster) |
