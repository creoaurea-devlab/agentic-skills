---
name: roadmap
description: >
  Planning and roadmap management skill with machine-readable YAML schemas for initiatives,
  projects, milestones, epics, sprints, and tasks. Use this skill when the user asks to:
  create a project plan, manage a roadmap, track progress, organize sprints, prioritize
  a backlog, estimate timelines, define milestones, or says anything like: "plan this",
  "roadmap", "what should we build next", "prioritize", "sprint planning", "milestone",
  "backlog", "project status", "what's left to do", or when organizing multi-week work
  into structured delivery phases.
---

# Roadmap

Flat-file, git-versioned roadmap management. One YAML file is the source of truth.
No external tools needed — agents and humans read the same file.

---

## Hierarchy

```
Initiative (strategic theme, e.g., "Platform Launch")
  └── Project (deliverable, e.g., "Auth System")
       └── Milestone (checkpoints with exit criteria)
            └── Epic (feature group)
                 └── Task (atomic work item, ≤1 day)
```

Each level has an `id`, `title`, `status`, and `owner`. Tasks also have `estimate` and `priority`.

---

## Quick Start

Create a `.project/roadmap.yaml` at the project root:

```yaml
roadmap:
  title: "My Project Roadmap"
  updated: 2026-03-18
  initiatives:
    - id: INIT-001
      title: "MVP Launch"
      status: in-progress
      projects:
        - id: PROJ-001
          title: "Core API"
          owner: "@team-backend"
          milestones:
            - id: MS-001
              title: "API v1 Complete"
              target_date: 2026-04-15
              exit_criteria:
                - "All CRUD endpoints passing integration tests"
                - "Auth middleware deployed"
              epics:
                - id: EPIC-001
                  title: "User Management"
                  tasks:
                    - id: T-001
                      title: "User registration endpoint"
                      status: done
                      estimate: 0.5d
                    - id: T-002
                      title: "User profile CRUD"
                      status: in-progress
                      estimate: 1d
                      priority: high
```

---

## Status Values

```
planned → in-progress → done
                      → blocked (requires note)
                      → cut (descoped, requires reason)
```

---

## Prioritization

Use WSJF (Weighted Shortest Job First) for backlog ordering:

```
WSJF = (business_value + time_criticality + risk_reduction) / job_size
```

| Factor | Scale |
|--------|-------|
| business_value | 1-5 |
| time_criticality | 1-5 |
| risk_reduction | 1-5 |
| job_size | 1-5 (1 = XS, 5 = XL) |

Higher WSJF → do first.

---

## Progress Computation

```
milestone_progress = done_tasks / total_tasks
project_progress = avg(milestone_progress for each milestone)
initiative_progress = avg(project_progress for each project)
```

Count only `done` tasks. `blocked` and `cut` tasks are excluded from the denominator
if they have `exclude_from_progress: true`.

---

## Agent Workflow for Updates

When asked to update the roadmap:

1. Read `.project/roadmap.yaml`
2. Identify which tasks/milestones changed
3. Update `status` and `updated` fields
4. Recompute progress for affected milestones/projects
5. If a milestone's exit criteria are all met → mark it `done`
6. Write the updated YAML back

When asked for status:

1. Read the roadmap
2. Compute progress at each level
3. Report: what's done, what's in progress, what's blocked, what's next

---

## Sprint Planning

For sprint-based teams, add a `sprints` section:

```yaml
sprints:
  - id: SPRINT-01
    title: "Sprint 1"
    start: 2026-03-18
    end: 2026-03-29
    goal: "Complete user registration flow"
    tasks: [T-001, T-002, T-003]
    velocity_planned: 8
    velocity_actual: null  # filled at sprint end
```

---

## Integration Points

- **Issue trackers** — export tasks as GitHub Issues / Linear tickets
- **CI/CD** — milestone exit criteria map to CI pipeline gates
- **Skills system** — the `implement` skill reads the roadmap to understand what to build next

For the full YAML schema definition with all fields and validation rules, read
`references/schemas.md`.

---

## Anti-Patterns

- **Roadmap without exit criteria** — milestones become meaningless checkboxes
- **Tasks larger than 1 day** — break them down; large tasks hide complexity
- **No owner on projects** — unowned work doesn't get done
- **Updating the tool but not the file** — the YAML file is the source of truth, always
- **Planning everything upfront** — plan one sprint ahead in detail, the rest in broad strokes
