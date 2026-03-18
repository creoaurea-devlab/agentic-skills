---
title: "SSRIV — Proposed Development Method"
status: experimental
stage: in-testing
version: "0.1.0"
author: "Kasparas Bagdonas / Creo Aurea"
created: "2026-03-18"
license: CC-BY-4.0
tags:
  - proposal
  - experimental
  - ai-assisted-development
  - anti-hallucination
  - architecture-pattern
---

# SSRIV — Proposed Development Method

> **⚠️ EXPERIMENTAL — IN TESTING**
> This is a proposed method, not a proven framework. It describes patterns observed during six months of building AI-assisted systems. It has not been validated at scale. Use it as a thinking tool, not a guarantee.

**Spec · Schema · Registry · Isolated Modules · Validation**

A five-layer pattern for structuring AI-assisted development to reduce hallucination, schema drift, and architectural decay. Proposed based on personal experience. Not peer-reviewed. Take it with appropriate skepticism.

---

## Why This Exists

After six months of building AI-assisted codebases — 20+ systems, ~50 prototypes — a pattern kept repeating:

- Projects start coherent and fast
- Past a certain size (roughly 8k–15k LOC), the AI starts guessing instead of knowing
- Schema drift appears: the implementation no longer matches what was intended
- Context windows fill up with irrelevant code; important contracts get forgotten
- The project stalls at 60–70% and never ships

The hypothesis: most of this is **structural**, not a model quality problem. Give the AI better inputs — cleaner scope, explicit contracts, a system map — and it produces better outputs.

SSRIV is an attempt to make that structure explicit and repeatable.

---

## The Core Idea

Five constraints, each closing one common failure mode:

```
Layer 1: SPEC       → "What are we building and why?"
Layer 2: SCHEMA     → "What shape does the data take?"
Layer 3: REGISTRY   → "What exists and where does it live?"
Layer 4: ISOLATION  → "What does this AI session see?"
Layer 5: VALIDATION → "Did execution match the intent?"
```

Each layer is meant to catch what the previous one misses.

**The key claim** (unverified): if you constrain an AI coding session to a single module, with its schema and neighbor interfaces visible but nothing else, it makes fewer mistakes and requires fewer corrections. The hypothesis is that focused context reduces hallucination surface.

---

## Layer 1 — Spec

Human intent in structured, unambiguous form.

Not just a comment or a prompt. A document with:
- What this builds (one sentence)
- Why it exists
- Explicit acceptance criteria (checkboxes)
- What is explicitly out of scope
- Which schemas it depends on

**Why bother**: Prose specs are ambiguous. An AI reading "handle authentication" might implement anything. A spec with explicit criteria and out-of-scope sections shrinks the interpretation space.

**Format tried**: Markdown with YAML frontmatter, stored in `.context/specs/`.

**What we don't know**: whether structured specs meaningfully reduce errors compared to well-written prompt comments. Needs more data.

---

## Layer 2 — Schema

Machine-parseable contracts as the single source of truth.

The pattern: define every input, output, entity, and error in Zod before writing any implementation. The schema becomes the contract — if the implementation drifts from it, `z.parse()` fails immediately.

```typescript
// Everything passes through Zod at boundaries
const result = SomeOutputSchema.parse(await runLogic(SomeInputSchema.parse(raw)))
```

**Why this helps**: it forces you to think about types before code, and creates a runtime enforcement mechanism that doesn't rely on remembering what the spec said.

**What schema-driven development alone misses**: it validates data shape but doesn't tell the AI what functions exist, what has been implemented, or whether the module behavior matches the spec's intent. That's what layers 3 and 5 try to address.

---

## Layer 3 — Registry

A self-describing map of the system: what features exist, what functions implement them, what their current status is.

```typescript
// Approximate shape — actual implementation varies
{
  id: "auth-login",
  status: "implemented" | "partial" | "stub" | "planned",
  functions: ["login", "refreshToken"],
  schemas: ["LoginInputSchema", "AuthResultSchema"],
  dependencies: ["database", "crypto"]
}
```

**The idea**: an AI that reads the registry before starting work knows what already exists, what is stubbed, and what its session is supposed to touch. It is less likely to invent functions that already exist or skip ones that need updating.

**Observed problem this addresses**: AI sessions in large codebases frequently hallucinate function names or call non-existent helpers. A validated registry makes that harder.

**What we don't know**: how much of the benefit comes from having the registry versus just having good file organization and clear import paths.

---

## Layer 4 — Isolated Modules

One AI session = one module. Visible context: the target module stub, its schema, and neighbor interfaces only.

```
Session sees:    target module + its schema + neighbor interfaces (as types, not implementations)
Session ignores: everything else in the codebase
```

This is enforced by how you structure the prompt, not by any tooling. The discipline is: never paste the whole codebase into context. Give the AI exactly what it needs for this one task.

**The observation**: sessions with narrow focused context produce code that requires fewer corrections and less rework than sessions with broad context. This pattern is consistent enough that it feels structural, not coincidental.

**The honest caveat**: this adds coordination overhead. You need module briefs, typed stubs, and interface files before you start implementing. For small projects, this is overkill.

---

## Layer 5 — Validation

The part most AI-assisted projects skip: execution-level proof that the implementation matches the intent.

Three mechanisms:

**Contract tests** — execute the function, parse the output against the declared schema:
```typescript
it("output conforms to schema", async () => {
  const result = await myFunction(validInput)
  expect(() => OutputSchema.parse(result)).not.toThrow()
})
```

**Spec acceptance tests** — turn spec acceptance criteria into executable assertions, not aspirational checkboxes.

**Drift detection** — automated checks for stale registry entries, unused schemas, stub functions that were never implemented.

**Why this matters**: without execution-level validation, the architecture exists on paper while the code drifts. Layers 1–4 structure the input to the AI. Layer 5 verifies the output matches what was intended.

---

## What SSRIV Does Not Solve

Being honest about limitations:

- **It does not fix a weak model.** If the AI fundamentally cannot implement what you need, structured inputs will not compensate.
- **It does not eliminate hallucinations.** It reduces the surface area and adds verification gates. Hallucinations still occur.
- **It adds upfront cost.** Writing specs, schemas, and registries before code takes time. For small one-off scripts, this is waste.
- **The validation layer requires discipline.** Contract tests only help if you write them and actually run them in CI.
- **It has not been tested on team codebases.** Everything here comes from solo development. Team dynamics are unknown.
- **The 60–70% stall problem.** SSRIV addresses the *architectural* causes of stalling (drift, context pollution, schema ambiguity). It does not address the *motivational* causes (ADHD, perfectionism, loss of momentum). Those are separate problems.

---

## Where It Came From

The patterns in SSRIV are not invented from theory. They emerged from building:

- A WebSocket AI workflow bridge (18 node types, multi-provider AI execution)
- A knowledge base CLI with TF-IDF search across 95 handles
- A node-based editor designed for ADHD workflows
- A code-indexing pipeline with Qdrant + Ollama
- 40+ other prototypes, many of which hit the same failure modes

The registry pattern came from getting burned twice by AI sessions inventing functions. The isolation pattern came from watching context windows fill with irrelevant code. The validation layer came from discovering that schemas pass at the boundary but the module behavior diverged three sessions later.

These patterns worked well enough that they became intentional. Whether they generalize to other people's projects is the open question.

---

## Comparison to Other Approaches

This is where I have the least certainty. The comparisons are rough:

| Approach | What it adds | What it misses |
|---|---|---|
| Vibe coding | Speed, flow | Structure, coherence at scale |
| Spec-driven (Kiro, OpenSpec) | Structured intent | Runtime contracts, system map |
| Schema-driven development | Runtime contracts | System map, isolation, execution proof |
| SSRI (4-layer predecessor) | Spec + Schema + Registry + Isolation | Execution validation gate |
| **SSRIV (this)** | All five layers | Proof that it works at scale |

The honest read: SSRIV is probably overkill for most projects. The full five-layer discipline makes sense at the scale where AI context starts becoming unreliable — somewhere past 8k–12k LOC in active development. Below that, good schema design and clear file organization get you most of the way.

---

## Status and What Comes Next

**Current status**: tested on personal projects within the OPEN-CLAW-BRIDGE workspace. Not tested externally. Not peer-reviewed. The patterns feel solid but the framework needs validation at scale with other people building other things.

**What would validate it**: someone outside this workspace tries SSRIV on a project between 10k–50k LOC, tracks errors by layer, and reports back. That data does not exist yet.

**What this document is**: a first ship. The ideas are real. The patterns have been practiced. The framework is proposed, not proven.

If you try it and it breaks, tell me. That is the point of shipping at 20%.

---

## Quick Reference

```
Before writing any code:
1. Write spec → .context/specs/feature.spec.md
2. Define schemas → .context/schemas/feature.schema.ts
3. Register the feature → .context/schemas/registry
4. Write typed stub → src/modules/feature/index.ts
5. One AI session = one module + its schema + neighbor interfaces
6. After implementing: contract test + spec acceptance test + drift check
7. Update lessons.md with anything unexpected
```

---

*SSRIV v0.1.0 — experimental proposal — Kasparas Bagdonas / Creo Aurea — 2026 — CC-BY-4.0*
