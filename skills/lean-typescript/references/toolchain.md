# Toolchain Reference — TypeScript 2026

## pnpm (Package Manager)

The 2026 standard. Faster than npm, less disk space than yarn, supports catalogs for monorepo version management.

```bash
pnpm init
pnpm add fastify zod
pnpm add -D typescript tsx vitest @playwright/test
```

### Catalogs (pnpm 9.5+)

Centralize versions in `pnpm-workspace.yaml` for monorepos:

```yaml
catalog:
  react: ^18.3.1
  zustand: ^5.0.0
  '@xyflow/react': ^12.0.0
```

Reference as `"catalog:"` in package.json. Enable `catalogMode: strict` to prevent out-of-catalog versions.

---

## tsx (Development Runner)

Run TypeScript directly — no build step during development.

```bash
pnpm add -D tsx
tsx src/index.ts          # Run once
tsx watch src/index.ts    # Watch mode with hot reload
```

---

## Vitest 4.0 (Testing)

23.5M weekly downloads, 10-20x faster than Jest in watch mode.

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      exclude: ['**/*.test.ts', '**/types.ts', 'dist/**'],
    },
  },
});
```

Key features:
- Browser mode with visual regression testing (stable in v4)
- Playwright trace integration
- `projects` mode replaces `vitest.workspace` for monorepos
- Built-in `expectTypeOf` for type-level testing
- `experimental.fsCache` for faster startup

---

## Biome 2.3 (Linting + Formatting)

Replaces ESLint + Prettier. 19-100x faster. Single `biome.json` config.

423+ lint rules. In benchmarks: 63ms vs ESLint's 1,200ms on 312 files.

```json
{
  "$schema": "https://biomejs.dev/schemas/2.3/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  }
}
```

Migration: `npx biome migrate eslint --write`

Complement with **Oxlint 1.0** (50-100x faster than ESLint, 695+ rules) as a fast pre-commit pass.

---

## tsconfig.json (Maximum Strict)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": true,

    "esModuleInterop": true,
    "resolveJsonModule": true,
    "isolatedModules": true,

    "outDir": "./dist",
    "declaration": true,
    "sourceMap": true,

    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Key flags explained:
- `noUncheckedIndexedAccess` — array/record access returns `T | undefined`, catches real null bugs
- `verbatimModuleSyntax` — forces `import type`, improves tree-shaking
- `exactOptionalPropertyTypes` — distinguishes `undefined` from "missing"
- `erasableSyntaxOnly` (5.8+) — enables Node.js type-stripping compatibility

Base package: `@tsconfig/strictest` bundles all of these.

---

## package.json Template

```json
{
  "name": "my-project",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest",
    "type-check": "tsc --noEmit",
    "lint": "biome check .",
    "format": "biome format --write ."
  }
}
```

---

## TypeScript 5.5-5.8 Features That Eliminate Boilerplate

- **Inferred type predicates (5.5)** — `.filter(x => x !== null)` now returns `T[]` not `(T | null)[]`
- **`satisfies` operator** — preserves literal types while enforcing structure
- **`using` keyword (5.2)** — automatic resource cleanup via `Symbol.dispose`
- **`erasableSyntaxOnly` (5.8)** — direct TS execution in Node.js 23.6+
- **TypeScript 7 (Go rewrite)** — 10-15x faster type-checking (coming)
