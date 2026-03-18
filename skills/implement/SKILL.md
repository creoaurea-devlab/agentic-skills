---
name: implement
description: >
  Agentic implementation skill that reads a project's structure, detects its tech stack,
  scores codebase health, and then selects the best strategies and sibling skills to
  implement the user's request — optimizing for smaller codebases and higher power-to-weight
  ratio. Use this skill whenever the user asks to: add a feature, refactor code, reduce
  bundle size, clean up a codebase, implement something new, improve code quality, reduce
  complexity, or says anything like "make this smaller", "make this better", "implement X",
  "build X", "clean this up", "reduce bloat", or "refactor". Also trigger when the user
  pastes code and asks for improvements, or when starting work on an unfamiliar project.
---

# Implement

Read first, think second, code last. Every implementation starts with understanding
what already exists — because the fastest code is the code you delete.

## Why This Skill Exists

LLMs default to additive solutions: more files, more abstractions, more dependencies.
This skill inverts that instinct. The goal is always: **fewer lines, fewer deps, more
capability**. Every change must justify its weight.

---

## Phase 1 — Read the Project (mandatory, never skip)

Before writing a single line, build a mental model of what exists.

### Step 1: Discover structure

Read these files in order of priority (skip what doesn't exist):

1. **Package manifest** — `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`
2. **Entry point** — `src/index.ts`, `src/main.ts`, `app/`, `cmd/`
3. **Config files** — `tsconfig.json`, `vite.config.*`, `.env.example`
4. **Context files** — `.context/`, `CLAUDE.md`, `AGENTS.md`, `README.md`
5. **Existing skills** — check for sibling skill folders in the skills directory

### Step 2: Detect tech stack

From the manifest and configs, identify:

- **Language & version** (TypeScript 5.x, Python 3.12, etc.)
- **Framework** (React, Next.js, Fastify, FastAPI, etc.)
- **State management** (Zustand, Redux, Pinia, etc.)
- **Validation** (Zod, Pydantic, Valibot, etc.)
- **Build tool** (Vite, esbuild, Turbopack, Ruff, etc.)
- **Test framework** (Vitest, pytest, Jest, etc.)

### Step 3: Score codebase health

Quick heuristics — no tooling needed:

| Signal | Healthy | Unhealthy |
|--------|---------|-----------|
| Largest file | <200 LOC | >500 LOC |
| Dependencies | <20 prod deps | >50 prod deps |
| `any` types | 0 | >5 |
| `utils/` or `helpers/` folders | absent | present |
| Default exports | 0 | >3 |
| Dead code signals | clean imports | unused imports visible |

Record findings mentally. These inform which strategies to apply.

---

## Phase 2 — Choose Strategy

Based on the project analysis, select the approach:

### Decision tree

```
Is this a new feature?
├── Yes → Check if existing module can absorb it (prefer extension over creation)
│         If new module needed → follow spec-driven skill (Spec → Schema → Module)
│         Apply lean-typescript patterns for implementation
│
Is this a refactor/cleanup?
├── Yes → Score health first
│         High bloat → apply lean-typescript debloating references
│         Architectural issues → apply spec-driven 4-pillar restructure
│         Dead code → run Knip or manual dead-export scan
│
Is this a bug fix?
├── Yes → apply debug-loop skill (reproduce → isolate → fix → verify)
│
Is this unclear?
├── Yes → ask the user one clarifying question, then proceed
```

### Sibling skill selection

Read the relevant sibling skill when the task matches:

- **`lean-typescript/`** — any TypeScript work, anti-bloat, patterns, toolchain
- **`spec-driven/`** — new modules, API contracts, schema design
- **`debug-loop/`** — bugs, startup failures, runtime errors
- **`roadmap/`** — planning, milestones, sprint organization

Load only what's needed. Don't read all skills for every task.

---

## Phase 3 — Implement

### The Lean Implementation Checklist

Before writing code, answer these questions:

1. **Can I solve this by deleting code?** If yes, do that instead.
2. **Can I extend an existing module?** Prefer modification over creation.
3. **Does this need a new dependency?** Only add if it saves >50 LOC and is <5KB gzipped.
4. **Is the file I'm creating <150 lines?** If not, split by domain concern.
5. **Am I adding types that TypeScript already infers?** Remove redundant annotations.
6. **Am I validating data that's already typed?** Validate at boundaries only.

### Implementation patterns (apply contextually)

- **Functions over classes** — unless the domain genuinely needs state + methods together
- **Named exports only** — better tree-shaking, better refactoring
- **Feature folders** — group by domain, not by technical layer
- **Result types over exceptions** — explicit error flow, no hidden control jumps
- **Zod/Pydantic at boundaries** — trust internal types, validate external data
- **Pure core, I/O shell** — business logic as pure functions, I/O at the edges

### What to reject

Stop and reconsider if you catch yourself:

- Creating a `utils.ts` or `helpers.ts`
- Wrapping every async call in try-catch
- Adding JSDoc that restates the type signature
- Building a class hierarchy (Manager → Service → Handler)
- Adding a config option for something that never changes
- Writing >20 lines of defensive null-checking inside typed code

---

## Phase 4 — Verify

After implementation, check your work:

1. **Line count delta** — did the codebase grow? By how much? Justify every new line.
2. **Dependency delta** — did you add deps? Each must pass the <5KB / >50LOC-saved test.
3. **Type safety** — run `tsc --noEmit` or equivalent. Zero errors.
4. **Tests** — run existing tests. If you changed behavior, update or add tests.
5. **Build** — verify the project builds cleanly.

### Completion summary

When done, report to the user:

```
## Changes
- [what changed, 1-2 sentences]

## Impact
- Files changed: N
- Lines added/removed: +X / -Y (net: ±Z)
- Dependencies added/removed: [list or "none"]

## Verified
- [ ] Types check
- [ ] Tests pass
- [ ] Build succeeds
```

---

## Anti-patterns

- **Implementing before reading** — you will duplicate existing functionality
- **Adding abstractions "for the future"** — YAGNI; implement what's needed now
- **Choosing a heavier tool** when a lighter one exists in the project
- **Ignoring existing patterns** — match the project's style, don't impose your own
- **Skipping verification** — unverified changes are unfinished changes
