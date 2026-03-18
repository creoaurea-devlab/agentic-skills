<p align="center">
  <h1 align="center">agentic-skills</h1>
  <p align="center">
    <strong>5 machine-readable skills that make AI coding agents write smaller, more powerful code.</strong>
  </p>
  <p align="center">
    <a href="#skills">Skills</a> · <a href="#research">Research</a> · <a href="#quick-start">Quick Start</a> · <a href="LICENSE">MIT License</a>
  </p>
</p>

---

## First Ship

This is my first public release. Six months of research, pattern-hunting, benchmarking libraries, testing architectures — and zero ships.

I have ADHD. I generate ideas fast and lose momentum when execution gets slow. I chase the perfect version and delay shipping when reality doesn't match the vision. Most of my projects get stuck at 80-95% because finishing is the hardest part.

This repo is me breaking that cycle. It's not perfect. It's shipped.

---

## What This Is

A collection of **agent skills** — portable instruction sets that AI coding agents load on demand to become better at specific tasks. They follow the [SKILL.md open standard](https://agentskills.io) and work across:

- **Claude Code** / Claude.ai
- **Windsurf** (Cascade)
- **Cursor**
- **Codex CLI**
- Any tool that reads SKILL.md files

The skills are opinionated toward **lean codebases**: fewer files, fewer dependencies, more capability per line. Every pattern in here was chosen because it makes code smaller without making it weaker.

---

## Skills

| Skill | What It Does | Lines |
|-------|-------------|-------|
| **[implement](skills/implement/SKILL.md)** | The brain — reads your project structure, detects tech stack, picks the best strategy, implements for minimal code | 131 |
| **[lean-typescript](skills/lean-typescript/SKILL.md)** | TypeScript 2026 anti-bloat rules, modern toolchain, architectural patterns | 144 + refs |
| **[spec-driven](skills/spec-driven/SKILL.md)** | Contract-first: Spec → Schema → Registry → Module | 134 |
| **[debug-loop](skills/debug-loop/SKILL.md)** | Autonomous debugging: reproduce → isolate → fix → prove with evidence | 129 |
| **[roadmap](skills/roadmap/SKILL.md)** | Planning with machine-readable YAML schemas for sprints, milestones, priorities | 130 + refs |

### How Skills Load

```
Level 1: Metadata        ~100 tokens   Always in context (name + description)
Level 2: SKILL.md body   <500 lines    Loaded when the skill triggers
Level 3: references/     Unlimited     Loaded on demand for deep dives
```

### How They Connect

```
implement (meta-skill)
├── reads your project first
├── selects the right skill:
│   ├── lean-typescript  → for any TypeScript work
│   ├── spec-driven      → for new modules or APIs
│   ├── debug-loop       → for bugs and failures
│   └── roadmap          → for planning tasks
└── verifies: did the codebase get smaller?
```

---

## Quick Start

### Option 1: Clone and point your agent at it

```bash
git clone https://github.com/CREOAUREAdevlab/agentic-skills.git
```

Then tell your AI coding agent:

> "Read the skills in `skills/implement/SKILL.md` and use them when implementing features."

### Option 2: Copy individual skills

Each skill is a self-contained folder. Copy any `skills/<name>/` folder into your project or your agent's skill directory.

### Option 3: Reference directly

Point your agent config at this repo. Most SKILL.md-compatible tools support loading skills from a path.

---

## Research

The [`research/`](research/) folder contains the curated research, benchmarks, and harvested patterns that informed these skills. Six months of investigating what actually works in 2025–2026 TypeScript development:

| Document | Contents |
|----------|----------|
| [TypeScript 2026 Stack](research/typescript-2026-stack.md) | Library choices with bundle sizes, from a 381-byte event bus to mutation testing |
| [TypeScript Patterns](research/typescript-patterns.md) | Anti-bloat commandments, architectural patterns, validation strategies |
| [Schema-Driven Dev](research/schema-driven-dev.md) | The 4-pillar methodology for contract-first, agent-safe development |
| [Roadmap Planning](research/roadmap-planning.md) | YAML roadmap schemas, RICE/WSJF prioritization, sprint workflows |

This is the "show your work" section — the data behind the recipes.

---

## Repo Structure

```
agentic-skills/
├── skills/
│   ├── implement/             # Agentic meta-skill
│   │   └── SKILL.md
│   ├── lean-typescript/       # TypeScript 2026 best practices
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── toolchain.md
│   │       ├── patterns.md
│   │       └── debloating.md
│   ├── spec-driven/           # Contract-first methodology
│   │   └── SKILL.md
│   ├── debug-loop/            # Autonomous debug workflow
│   │   └── SKILL.md
│   └── roadmap/               # Planning & roadmap management
│       ├── SKILL.md
│       └── references/
│           └── schemas.md
├── research/                  # Source research & benchmarks
│   ├── README.md
│   ├── typescript-2026-stack.md
│   ├── typescript-patterns.md
│   ├── schema-driven-dev.md
│   └── roadmap-planning.md
├── .gitignore
├── CONTRIBUTING.md
├── LICENSE                    # MIT
└── README.md                  # You are here
```

---

## Design Principles

- **Concise** — only include what the agent doesn't already know
- **Progressive disclosure** — metadata → body → references (loaded on demand)
- **Explain the why** — reasoning over rigid rules (LLMs are smart)
- **Agent/IDE agnostic** — standard format, no vendor lock-in
- **Ship > perfect** — done beats theoretical

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs welcome — especially new skills, pattern improvements, or research additions.

---

## License

[MIT](LICENSE) — use it, fork it, ship it.

---

<p align="center">
  <sub>Built by <a href="https://github.com/CREOAUREAdevlab">@CREOAUREAdevlab</a> — first ship, not last.</sub>
</p>
