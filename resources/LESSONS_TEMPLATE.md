# Engineering Lessons & Post-Mortem Memory (`LESSONS.md`)

> **Purpose**: AI agents start every session with zero memory of past bugs. This file is the project's institutional memory.
> **Mandatory Rule**: Every agent MUST read this file before starting work and MUST append a new entry whenever a bug, test failure, or architectural gotcha is resolved.

---

## Active Project Invariants (Never Violate)

1. **Contract Truth**: `docs/em/BRIEF.md` defines the system's Input/Output boundaries. Never alter external I/O signatures without updating the Brief and getting Manager approval.
2. **Quality Gate**: No task is complete until `make verify` passes with 0 errors and 0 warnings.
3. **Test Integrity**: Tests are sacred. Never weaken, skip, or delete a failing test to make CI green.

---

## Outage & Bug Post-Mortem Log

### Template for New Entries
```markdown
### [YYYY-MM-DD] <Short Title of Bug / Outage / Gotcha>
- **Symptom**: What failed or behaved unexpectedly?
- **Root Cause**: Why did the agent or system make this mistake?
- **Fix Applied**: How was it solved?
- **Prevention Rule**: What specific rule or automated test prevents this from ever happening again?
```

---

### Logged Lessons
*(Append chronological lessons below)*
