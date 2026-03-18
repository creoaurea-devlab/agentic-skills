# Roadmap YAML Schema Reference

Complete field definitions for `.project/roadmap.yaml`.

---

## Top-Level Structure

```yaml
roadmap:
  title: string            # Project or product name
  updated: date            # ISO date of last update (YYYY-MM-DD)
  version: string          # Schema version (e.g., "1.0")
  initiatives: Initiative[]
  sprints?: Sprint[]       # Optional sprint-based planning
```

---

## Initiative

```yaml
- id: string               # Format: INIT-NNN
  title: string
  status: Status
  description?: string     # Strategic context (1-2 sentences)
  owner?: string           # Team or person (@handle)
  projects: Project[]
```

## Project

```yaml
- id: string               # Format: PROJ-NNN
  title: string
  status: Status
  owner: string            # Required — unowned projects don't ship
  description?: string
  milestones: Milestone[]
  risks?: Risk[]
```

## Milestone

```yaml
- id: string               # Format: MS-NNN
  title: string
  status: Status
  target_date: date        # YYYY-MM-DD
  exit_criteria: string[]  # All must be true to mark "done"
  epics: Epic[]
```

## Epic

```yaml
- id: string               # Format: EPIC-NNN
  title: string
  status: Status
  description?: string
  tasks: Task[]
```

## Task

```yaml
- id: string               # Format: T-NNN
  title: string
  status: Status
  estimate: string         # Duration: "0.5d", "1d", "3h"
  priority: Priority
  owner?: string
  blocked_by?: string[]    # Task IDs this depends on
  blocked_reason?: string  # Required when status is "blocked"
  cut_reason?: string      # Required when status is "cut"
  exclude_from_progress?: boolean  # true for blocked/cut tasks to exclude from %
```

---

## Enums

### Status

```yaml
enum Status:
  - planned         # Not started
  - in-progress     # Actively being worked on
  - done            # Complete
  - blocked         # Waiting on dependency (requires blocked_reason)
  - cut             # Descoped (requires cut_reason)
```

### Priority

```yaml
enum Priority:
  - critical        # Blocks other work, must be done now
  - high            # Important for current milestone
  - medium          # Should be done this sprint
  - low             # Nice to have, do if time permits
```

---

## Sprint

```yaml
- id: string               # Format: SPRINT-NN
  title: string
  start: date
  end: date
  goal: string             # 1-sentence sprint objective
  tasks: string[]          # Task IDs assigned to this sprint
  velocity_planned: number # Story points or task count planned
  velocity_actual?: number # Filled at sprint end
  retrospective?: string   # Brief notes on what went well/badly
```

---

## Risk

```yaml
- id: string               # Format: RISK-NNN
  title: string
  severity: high | medium | low
  probability: high | medium | low
  impact: string           # What happens if this risk materializes
  mitigation: string       # What we're doing to prevent/reduce it
  status: open | mitigated | realized
```

---

## WSJF Prioritization Fields (optional per-task)

```yaml
  wsjf:
    business_value: 1-5
    time_criticality: 1-5
    risk_reduction: 1-5
    job_size: 1-5
    # Computed: score = (bv + tc + rr) / js
```

---

## Progress Computation Rules

```
task_done = status == "done"
task_counted = status != "cut" OR exclude_from_progress != true

milestone_progress = count(task_done) / count(task_counted)
project_progress = mean(milestone_progress for each milestone)
initiative_progress = mean(project_progress for each project)
```

---

## Stakeholder Export Formats

### Status summary (for reporting)

```yaml
status_report:
  date: 2026-03-18
  initiative: "MVP Launch"
  overall_progress: 0.65
  on_track: true
  highlights:
    - "User registration complete"
    - "Auth middleware deployed"
  blockers:
    - task: T-005
      reason: "Waiting on Stripe API approval"
  next_sprint:
    - "Profile CRUD"
    - "Password reset flow"
```

### Changelog (for release notes)

```yaml
changelog:
  version: "0.2.0"
  date: 2026-03-18
  added:
    - "User registration endpoint"
    - "Email verification flow"
  changed:
    - "Auth tokens now expire after 24h (was 7d)"
  fixed:
    - "Password validation accepting empty strings"
```
