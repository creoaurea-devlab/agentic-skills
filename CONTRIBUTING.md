# Contributing

Thanks for considering a contribution! This project is open to improvements of all kinds.

## Ways to contribute

- **New skills** — add a `skills/<name>/SKILL.md` following the existing format (YAML frontmatter + markdown body under 500 lines)
- **Improve existing skills** — fix errors, add patterns, improve clarity
- **Research additions** — add benchmarks, comparisons, or new findings to `research/`
- **Bug reports** — if a skill gives bad advice, open an issue

## Skill format

Every skill must have:

1. A folder in `skills/` with at least a `SKILL.md`
2. YAML frontmatter with `name` (lowercase-hyphenated, max 64 chars) and `description` (max 1024 chars)
3. Body under 500 lines — use `references/` for longer content
4. Agent/IDE-agnostic instructions (no vendor-specific commands)

## Style

- Be concise — every token costs context window space
- Explain *why*, not just *what* — LLMs respond better to reasoning than rigid rules
- Test with a real AI agent before submitting
- No secrets, API keys, or personal paths in any file

## Process

1. Fork the repo
2. Create a branch (`git checkout -b add-my-skill`)
3. Make your changes
4. Open a PR with a clear description of what the skill does and why

That's it. No CI to pass, no tests to run (yet). Ship it.
