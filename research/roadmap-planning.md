# Roadmap & Planning Systems

> Machine-readable YAML roadmap schemas, prioritization frameworks, sprint planning, and agent-executable workflows for project management.

---

## Core Principle

The roadmap lives in `.project/roadmap.yaml`. It is:
- **Machine-readable** — agents query and update it via tools
- **Human-readable** — valid YAML with clear prose fields
- **Git-versioned** — every change is a commit, fully auditable
- **Linked** — references ticket IDs from the planning system

---

## Roadmap Hierarchy

```
INITIATIVE (Business objective, quarters)
  └── PROJECT (Product area, months)
        └── MILESTONE (Shippable release, weeks)
              └── EPIC (Feature cluster, days)
                    └── STORY / TASK (Implementation, hours)
```

---

## Full Roadmap Schema

```yaml
# .project/roadmap.yaml
version: "1.0"
updated: "2026-03-16T14:00:00Z"
updated_by: "@orchestrator"

project:
  name: "My Product"
  vision: "One-line statement of where we're heading"
  current_quarter: "Q2-2026"

initiatives:
  - id: INIT-001
    title: "Platform Launch"
    description: "Core platform ready for public access"
    status: in-progress       # planned | in-progress | complete | paused | cancelled
    owner: "@cto"
    horizon: "2026-Q2"
    okrs:
      - "Achieve 1,000 active users by end of Q2"
      - "100% uptime SLA for core auth and billing"
    projects:
      - id: PROJ-001
        title: "User Management"
        status: in-progress
        lead: "@alice"
        progress: 0.45        # 0.0–1.0, auto-computed or manual
        milestones:
          - id: MS-001
            title: "Auth Complete"
            target: "2026-04-15"
            status: in-progress
            epics: [EPIC-001, EPIC-002]
            deliverables:
              - "OAuth2 Google + GitHub login"
              - "Session management + logout"
              - "Password reset flow"

releases:
  - id: v1.0-alpha
    title: "Alpha Release"
    target: "2026-04-15"
    status: planned
    milestones: [MS-001]

current_sprint:
  id: SPRINT-12
  start: "2026-03-11"
  end: "2026-03-25"
  goal: "Complete OAuth2 integration and session management"
  capacity_points: 40
  committed_points: 38
  completed_points: 22
  velocity_last_3: [34, 38, 41]

risks:
  - id: RISK-001
    title: "OAuth provider rate limits during load testing"
    probability: medium
    impact: high
    mitigation: "Use mock OAuth server for load tests"
    status: open
    affects: [MS-001]
```

---

## Sprint Schema

```yaml
# .project/sprints/SPRINT-12.yaml
id: SPRINT-12
name: "Sprint 12 — Auth & Session"
start: "2026-03-11"
end: "2026-03-25"
status: active            # planning | active | review | complete
goal: "Deliver complete OAuth2 login and session management"

team:
  capacity_points: 40
  members:
    - id: "@dev-1"
      type: agent
      capacity_points: 16
    - id: "@alice"
      type: human
      capacity_points: 8

committed:
  - task_id: TASK-101
    points: 5
    assignee: "@dev-1"
  - task_id: TASK-102
    points: 3
    assignee: "@dev-1"

metrics:
  committed_points: 38
  completed_points: 22
  carry_over_points: 0
  bugs_found: 2
  bugs_fixed: 1

retrospective: null       # filled at sprint end
```

---

## Milestone Schema

```yaml
# .project/milestones/MS-001_auth-complete.yaml
id: MS-001
title: "Auth Complete"
description: "All authentication flows working in production"
status: in-progress
target: "2026-04-15"
owner: "@alice"

epics: [EPIC-001, EPIC-002]
depends_on: []
blocks: [MS-002, MS-003]

success_criteria:
  - "OAuth2 Google + GitHub login working E2E"
  - "Session persistence across browser restarts"
  - "Password reset email delivery < 30s"
  - "Zero P0 auth bugs in staging for 72h"

definition_of_done:
  - "All acceptance criteria passed"
  - "QA sign-off"
  - "Security review completed"
  - "Monitoring dashboards live"
  - "Runbook written"

progress:
  total_tasks: 24
  done_tasks: 11
  in_progress_tasks: 6
  blocked_tasks: 1
  completion_pct: 0.46

health: yellow            # green | yellow | red
health_reason: "TASK-100 blocked on OAuth provider setup"
```

---

## Prioritization Frameworks

### RICE (Recommended default)

```
RICE = (Reach x Impact x Confidence) / Effort

Reach:      users affected per quarter
Impact:     0.25=minimal | 0.5=low | 1=medium | 2=high | 3=massive
Confidence: 0.0–1.0
Effort:     person-weeks to implement
```

| Feature | Reach | Impact | Confidence | Effort | RICE |
|---------|-------|--------|------------|--------|------|
| OAuth Login | 1000 | 3 | 0.9 | 3 | 900 |
| Notifications | 800 | 2 | 0.8 | 4 | 320 |
| Dark Mode | 600 | 1 | 0.9 | 2 | 270 |

### WSJF (Alternative)

```
WSJF = (business_value + time_criticality + risk_reduction) / job_size
```

### MoSCoW (For release scoping)

```yaml
release: v1.0-beta
must_have:    ["OAuth2 login", "Session management", "RBAC"]
should_have:  ["Password reset", "User profile edit"]
could_have:   ["2FA support", "Dark mode"]
wont_have:    ["Mobile native app", "Offline mode"]
```

---

## Progress Computation

```python
def compute_milestone_progress(milestone_id: str) -> float:
    tasks = query_tasks(milestone=milestone_id)
    done = [t for t in tasks if t.status in ("done", "merged")]
    return len(done) / len(tasks) if tasks else 0.0

def forecast_completion(milestone_id: str) -> date:
    remaining_points = sum(t.estimate for t in get_open_tasks(milestone_id))
    velocity = get_team_velocity(last_n_sprints=3)
    return today + timedelta(days=remaining_points / velocity)
```

---

## Agent Workflow for Updates

```
1. task.done event fires
2. Agent calls update_task(id, status="done")
3. Recompute progress for parent story
4. If story complete → mark done → recompute parent epic
5. If epic complete → mark done → update roadmap.yaml progress
6. If all milestone epics done → flag for human review
7. Emit roadmap.updated event
```

---

## Stakeholder Export: One-Pager

```markdown
## Q2-2026 Roadmap Summary

**Goal:** Platform Launch (INIT-001)

| Milestone     | Target  | Status        | Progress |
|---------------|---------|---------------|----------|
| Auth Complete | Apr 15  | In progress   | 46%      |
| Permissions   | May 30  | Planned       | 0%       |
| Dashboard MVP | Jun 1   | Planned       | 0%       |

**At risk:** MS-001 forecasts Apr 22 (+7 days slip)
**Blocker:** OAuth provider setup delayed
```

---

## Anti-patterns

- **Roadmap as slide deck only** — always have the YAML as source of truth; slides are exports
- **Progress updated manually** — automate from ticket states
- **Milestones without success criteria** — agents cannot verify completion without them
- **No dependency links** — orchestrator cannot sequence work without `depends_on`
