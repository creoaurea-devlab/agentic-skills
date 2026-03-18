# Schema-Driven Development

> A contract-first, agent-safe development methodology where every unit of work is defined before it's built, validated at runtime, and discoverable without reading source code.

---

## The 4 Pillars

### 1. SPEC — Intent Layer

Human-readable contract describing what a module does, its inputs, outputs, side effects, and dependencies.

- Lives in `.context/specs/[module].md` or inline YAML frontmatter
- Written **before** code — acts as the prompt for AI agents AND the acceptance criteria for humans
- Answers: *"What should exist and why?"*

### 2. SCHEMA — Validation Layer

Machine-enforceable shape of all data using Zod (TypeScript), Pydantic (Python), or JSON Schema.

- Covers: API payloads, config objects, DB records, inter-module messages
- Generates: TypeScript types, OpenAPI docs, mock fixtures
- Answers: *"What shape is the data, and is it valid right now?"*

### 3. REGISTRY — Discovery Layer

Central manifest listing all modules, their spec references, schema IDs, and dependency graph.

- Enables AI agents to enumerate capabilities without filesystem crawling
- Enables humans to audit scope at a glance
- Enables CLI tools to scaffold from known patterns
- Answers: *"What exists, where, and how does it connect?"*

### 4. ISOLATED MODULES — Execution Layer

Each module has a single responsibility, owns its schema, and exposes a typed interface.

- No implicit globals — all dependencies injected (Awilix, DI container, or plain function params)
- Independently testable, replaceable, and AI-generatable from spec alone
- Answers: *"What is this unit, and can I swap it without side effects?"*

---

## The Flow

```
SPEC (intent) → SCHEMA (contract) → REGISTRY (register) → MODULE (implement)
     ↑                                                            |
     └──────────── feedback loop (schema drift → update spec) ───┘
```

---

## Why It Works for AI-Assisted Development

| Problem | Solution |
|---------|----------|
| AI hallucinates module APIs | Schema = ground truth |
| AI doesn't know what exists | Registry = enumerable map |
| AI overwrites unrelated code | Isolated modules = blast radius contained |
| Human loses context mid-sprint | Spec = re-entry point |

---

## Practical Artifacts Per Module

```
.context/
  specs/cart.md          ← SPEC
  schemas/cart.zod.ts    ← SCHEMA
  registry.json          ← REGISTRY entry

src/modules/cart/
  index.ts               ← isolated module (public API)
  cart.test.ts           ← tests driven by spec
```

---

## TL;DR

Write the contract first. Validate the shape. Register the existence. Isolate the implementation.

Agents and humans can both operate safely at any layer.
