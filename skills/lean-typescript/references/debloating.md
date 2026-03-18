# Debloating Reference

Concrete library swaps and tools for reducing bundle size and codebase weight.

---

## Library Replacements

### lodash → es-toolkit (97% smaller)

Drop-in replacement. Used by Storybook, MUI, Recharts. 2-3x faster.

| Function | es-toolkit | lodash-es | Reduction |
|----------|-----------|-----------|-----------|
| sample | 88 bytes | 2,000 bytes | 96% |
| omit | ~200 bytes | ~5 KB | 96% |
| difference | ~100 bytes | ~3 KB | 97% |

```typescript
// Before
import { pick } from 'lodash-es';
// After
import { pick } from 'es-toolkit';
```

For functional composition (pipes, lazy evaluation), pair with **Remeda** — TypeScript-first, data-first and data-last APIs, 9.08KB gz full library, tree-shakable.

### Zod v3 → Zod v4 Mini (~10-15KB savings)

Zod v4 is a ground-up rewrite: 14x faster string parsing, 7x faster array parsing.

`zod/v4-mini`: functional, tree-shakable API at ~1.88KB gz (via Rolldown).

| Library | Typical schema (gz) | Tree-shakable |
|---------|-------------------|---------------|
| Zod v3 | ~15-17 KB | Poor |
| Zod v4 Mini | ~1.88 KB | Excellent |
| Valibot | ~1.37 KB | Excellent |

Zod v4 wins on ecosystem: `.toJSONSchema()`, tRPC integration, React Hook Form resolvers.

### moment.js / dayjs → Temporal API (0KB)

Temporal API landed in Chrome 144 (Jan 2026) and Firefox 139. For Electron/modern Node:

```typescript
const today = Temporal.Now.plainDateISO();
const deadline = today.add({ days: 14 });
const duration = Temporal.Duration.from({ hours: 2, minutes: 30 });
```

Saves: moment.js (~18KB gz), Luxon (~23KB gz), or dayjs (~2KB gz).

### Full Effect-TS → neverthrow (~8KB savings)

If you only need Result types (not the full fiber runtime):

```typescript
import { ok, err, ResultAsync } from 'neverthrow';

const divide = (a: number, b: number) =>
  b === 0 ? err('division by zero') : ok(a / b);

// Chain with andThen, map, mapErr
divide(10, 2).map(v => v * 100);  // Ok(500)
```

Even lighter: `@praha/byethrow` (<1KB gz) using plain objects.

---

## Dead Code Elimination

### Knip

Gold standard. Supersedes ts-prune, unimported, depcheck. Finds unused files, exports, dependencies, and enum members.

```bash
pnpm add -D knip && npx knip
```

- 80+ framework plugins (React, Vite, Vitest, etc.)
- Run `--production` mode to audit only production entry points
- At Vercel, Knip helped delete ~300,000 lines of unused code

Run in CI as a required check.

### Biome noUnusedImports

Real-time detection during development. Catches unused imports per-file.

### Bundle Analysis

```bash
pnpm add -D rollup-plugin-visualizer
```

Add to Vite config for interactive treemaps showing exactly what's in your bundle.

---

## Build Optimization Checklist

1. **`build.target: 'esnext'`** — skip transpilation if targeting modern runtime
2. **`base: './'`** — proper file:// protocol loading (Electron)
3. **`manualChunks`** — split heavy vendor deps (XYFlow, etc.)
4. **Lazy loading** — dynamic `import()` for heavy components
5. **`sideEffects: false`** in package.json for barrel files
6. **Tree-shake barrel exports** — or better, avoid barrels internally

---

## Estimated Savings from Full Debloating

| Swap | Savings (gzipped) |
|------|--------------------|
| lodash → es-toolkit | ~20-24 KB |
| Zod v3 → Zod v4 Mini | ~10-15 KB |
| moment.js → Temporal | ~18 KB |
| Effect-TS → neverthrow | ~5-10 KB |
| Knip dead code removal | 10-30% additional |
| **Total typical** | **~50-70 KB + dead code** |
