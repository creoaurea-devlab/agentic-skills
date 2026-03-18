---
name: debug-loop
description: >
  Autonomous debugging workflow: reproduce the failure, isolate the highest-leverage
  blocker, fix it, rerun, and prove success with runtime evidence. Use this skill when
  the user reports a bug, a test failure, a crash, a startup error, a build failure, or
  says anything like: "it's broken", "not working", "error", "crash", "fix this", "debug",
  "why does this fail", "help me find the bug", "it worked before", or when you encounter
  an error during implementation that needs systematic debugging. Also use when launching
  an app for the first time and verifying it runs correctly.
---

# Debug Loop

Prove the root cause before fixing. Every debug session follows the same loop until
the problem is solved or you run out of hypotheses to test.

---

## The Loop

```
1. CONTEXT    → Read error messages, logs, recent changes
2. REPRODUCE  → Run the exact command/action that fails
3. ISOLATE    → Find the single highest-leverage blocker
4. FIX        → Apply the minimal fix (one change at a time)
5. RERUN      → Run the exact same command/action again
6. PROVE      → Show runtime evidence of success
```

Repeat steps 3-6 until all blockers are resolved. Never skip REPRODUCE or PROVE.

---

## Phase 1: Context

Before touching code, gather information:

1. **Read the error** — the full stack trace, not just the first line
2. **Check recent changes** — `git diff HEAD~3` or recent file modifications
3. **Read the config** — package.json scripts, tsconfig, env files
4. **Check dependencies** — `node_modules` exists? Lock file matches?

The error message usually tells you where to look. Read it carefully.

---

## Phase 2: Reproduce

Run the **exact command** the user ran (or the exact action that triggers the bug).

```bash
# Whatever they told you, run it verbatim first
pnpm dev
pnpm test
pnpm build
```

If it doesn't reproduce:
- Check environment differences (Node version, OS, env vars)
- Ask for exact reproduction steps
- Try `rm -rf node_modules && pnpm install` for dependency issues

**You cannot fix what you cannot reproduce.**

---

## Phase 3: Isolate

You likely see multiple errors. Don't fix them all at once. Find the **first domino**:

### Priority ladder

1. **Missing dependency / import error** — nothing works until this is fixed
2. **Type error at build time** — prevents compilation
3. **Runtime crash on startup** — app never reaches functional state
4. **Runtime error on action** — specific feature is broken
5. **Wrong behavior** — works but produces incorrect results
6. **Performance issue** — works correctly but too slow

Fix from the top. Lower-priority issues often disappear when higher ones are resolved.

### Isolation technique

When the root cause isn't obvious:

- **Binary search** — comment out half the code, does it still fail?
- **Minimal reproduction** — extract the failing code into a standalone file
- **Dependency elimination** — does it fail with mocked dependencies?
- **Git bisect** — which commit introduced the regression?

---

## Phase 4: Fix

Apply the **minimal fix** that addresses the root cause:

- Fix the cause, not the symptom
- Change one thing at a time
- Prefer upstream fixes over downstream workarounds
- If the fix is >10 lines, pause and reconsider — you might be fixing the wrong thing

### Common fix patterns

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `Cannot find module` | Missing dep or wrong import path | `pnpm add X` or fix path |
| `TypeError: X is not a function` | Wrong import (default vs named) | Check the library's export |
| `EADDRINUSE` | Port already occupied | Kill existing process or change port |
| `Type 'X' is not assignable` | Schema/type mismatch after change | Update the type or the usage |
| `CORS error` | Server doesn't allow origin | Add CORS headers server-side |
| `Module not found` in build | Missing from `dependencies` (in `devDependencies`) | Move to correct section |

---

## Phase 5: Rerun

Run the **exact same command** from Phase 2. Not a different one. Not a subset.

If the original error is gone but a new one appears → go back to Phase 3 (isolate the new blocker).

If the error persists → your fix was wrong. Revert it. Go back to Phase 3 with new information.

---

## Phase 6: Prove

Show **runtime evidence** that the fix works:

- Terminal output showing clean startup
- Test output showing green passes
- Browser/app screenshot showing the feature working
- HTTP response showing correct data

"I think it should work now" is not proof. Run it and show the output.

---

## Launch Verification (for first-time startup)

When launching an app for the first time, the debug loop becomes a verification loop:

1. Install dependencies (`pnpm install`)
2. Check for missing env vars (`.env.example` vs `.env`)
3. Run build (`pnpm build`) — fix any type/compile errors
4. Run dev server (`pnpm dev`) — verify it starts
5. Hit the main endpoint/UI — verify it responds
6. Run tests (`pnpm test`) — verify test suite passes

Each step that fails enters the debug loop until resolved.

---

## Separate Launcher from Runtime Bugs

**Launcher bugs** prevent the app from starting at all. Fix these first:
- Missing env vars, wrong Node version, missing deps, port conflicts, bad config

**Runtime bugs** appear after startup. Fix these second:
- Broken features, wrong data, UI issues, API errors

Never debug runtime issues if the app won't even start.

---

## Handoff Summary

When debugging is complete, report:

```
## Debug Summary
- **Problem**: [1-sentence description of the original issue]
- **Root cause**: [what actually caused it]
- **Fix**: [what you changed, with file paths]
- **Evidence**: [command + output proving it works]
- **Regression risk**: [low/medium/high + what to watch for]
```

---

## Anti-Patterns

- **Fixing without reproducing** — you're guessing, not debugging
- **Fixing multiple things at once** — you won't know which fix worked
- **Downstream workarounds** instead of upstream fixes — masks the real problem
- **Reading code instead of running code** — theories need runtime evidence
- **Ignoring the stack trace** — it literally tells you where the bug is
- **"It works on my machine"** — reproduce in the exact failing environment
