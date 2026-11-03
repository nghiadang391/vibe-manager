---
name: vibe-manager
description: >-
  Engineering Manager (EM) and Tech Lead orchestration workflow for managing AI coding
  agents across any programming domain (Web, Backend, Blockchain, SystemC/Hardware modeling,
  Embedded, Mobile, AI/Data). Activate whenever the user wants to manage a project at a
  high level without reading raw code or PRs, asks for an EM/Tech Lead brief, invokes
  "vibe manager", or asks for high-level architecture, input/output contracts, AI guardrails,
  automated verification harnesses, and subagent code audits.
---

# Vibe Manager: Engineering Manager & Tech Lead Orchestration Skill

This skill shifts the human user from a **Bricklayer** (reading/writing every line of code and reviewing raw PRs) to an **Engineering Manager (EM) & Tech Lead** (approving blueprints, enforcing automated quality gates, and auditing outcomes through independent subagents).

It applies to **any programming field** (Web, Blockchain, SystemC/RTL modeling, Embedded C/C++, Distributed Systems, Data/ML).

---

## Core Lifecycle Checklist (Phase-Gated Workflow)

Never jump straight from a user prompt to writing feature code. Follow these 5 gates in strict order:

```text
[ ] Phase 1: Domain Research & Visual 1-Pager Brief  -> Present high-level flow, I/O, stack, and top 3 risks
[ ] Phase 2: Manager Approval Gate                   -> Wait for explicit user approval on Brief & Decisions
[ ] Phase 3: Project Memory & Anti-Blind-AI Rules    -> Initialize `docs/em/` and `.agents/rules/`
[ ] Phase 4: Automated Verification Harness (Step 0) -> Build runnable `make verify` + CI config before features
[ ] Phase 5: Builder + Reviewer Subagent Loop        -> Build feature, audit via fresh subagent, deliver Sign-Off Report
```

---

## Phase 1: Domain Research & Visual 1-Pager Brief

First, detect whether the workspace is a **New Project (Greenfield)** or an **Old / Existing Workspace (Brownfield)**:

- **Mode A — New Project (Empty or Skeleton Repo)**:
  - Consult [DOMAIN_HARNESS_MATRIX.md](./references/DOMAIN_HARNESS_MATRIX.md) (and web search if specialized) to propose the ideal stack, architecture, I/O contracts, and verification harness.
- **Mode B — Old / Existing Workspace (Legacy or Active Repo)**:
  - **Do NOT propose rewriting the stack from scratch.** First scan the repository structure, build files (`package.json`, `CMakeLists.txt`, `Makefile`, `Cargo.toml`, `foundry.toml`), existing tests, and CI configs (delegate to the `research` subagent if the codebase is large).
  - Reverse-engineer the **actual current architecture and Input/Output flows** so the Manager gets an instant **Executive Onboarding Brief** without reading the old code.
  - Run a **Current Health & Gap Audit**: identify what already works vs. what guardrails are missing (e.g., missing test coverage, unconfigured linters, undocumented I/O, or high-risk legacy modules).

Next, generate the **Visual 1-Pager Brief** using [BRIEF_TEMPLATE.md](./resources/BRIEF_TEMPLATE.md). The brief **MUST** include:
1. **Domain Mental Model (General Knowledge)**: 3–5 plain-English bullet points explaining how this technology domain works, key concepts, and industry-standard terminology so the Manager has full context without reading low-level docs.
2. **High-Level Architecture & Flow**: A Mermaid diagram (`flowchart LR` or `sequenceDiagram`) showing component boundaries and data/signal flow (existing flow + proposed changes if in an old workspace).
3. **Strict Input / Output Contract Table**: Every external input, output, type/protocol, validation rule, and failure behavior.
4. **Tech Stack & Current Health Audit**:
   - *New Project*: Recommended stack with 1-line justification.
   - *Old Workspace*: Detected stack + **Current Guardrail Status** (`PASS` / `MISSING` / `BROKEN` for build, linter, tests, and CI).
5. **Important High-Level Points (Top 3–5 Risks & Trade-Off Decisions)**: Highlight critical architectural bottlenecks, legacy tech debt risks, security gaps, or domain trade-offs that the Manager must care about.
6. **Verification & Staging Plan**: How the system will be automatically tested (linters, unit/simulation tests, coverage target, CI pipeline, and staging/simulation environment).

---

## Phase 2: Manager Approval Gate (Stop & Ask)

**CRITICAL RULE**: Do **NOT** write feature code during Phase 1 or Phase 2.

1. Present the **Visual 1-Pager Brief** to the user.
2. Explicitly ask the user to confirm or choose their preference on the **Top 3–5 Risks/Trade-Off Decisions** (use `ask_question` if choices are discrete).
3. Wait for the user's explicit **"Approved"** before proceeding to Phase 3.

---

## Phase 3: Project Memory & Anti-Blind-AI Rules Setup

Once the Manager approves the Brief, install persistent management memory and behavioral guardrails into the project so agents never act like blind code-monkeys across sessions:

1. **Create Management Directory (`docs/em/`)**:
   - `docs/em/BRIEF.md`: Save the approved Visual 1-Pager Brief (the living source of truth for system I/O and architecture).
   - `docs/em/LESSONS.md`: Initialize from [LESSONS_TEMPLATE.md](./resources/LESSONS_TEMPLATE.md) if it does not exist. This acts as the team's **Post-Mortem & Outage Memory**.

2. **Install AI Behavioral Rules (`.agents/rules/em-guardrails.md`)**:
   - Create `.agents/rules/em-guardrails.md` from [AGENT_RULES_TEMPLATE.md](./resources/AGENT_RULES_TEMPLATE.md).
   - These rules enforce four non-negotiable behaviors:
     - **Rule 1 — Mandatory Architecture Pushback**: If a user request or implementation shortcut introduces tight coupling, spaghetti state, security flaws, or breaks `docs/em/BRIEF.md`, the agent **must stop and push back** with a cleaner alternative before writing code. Blind agreement is a failure.
     - **Rule 2 — Outage & Bug Memory (`LESSONS.md`)**: Before starting any task, read `docs/em/LESSONS.md`. Whenever a test fails unexpectedly, a bug is fixed, or a domain gotcha is discovered, append a root-cause entry to `docs/em/LESSONS.md`.
     - **Rule 3 — Pre-Mortem Edge-Case Table**: Before implementing any feature, list at least 5 edge cases (boundary values, malformed inputs, race/timing conditions, resource exhaustion) and map each to an automated test.
     - **Rule 4 — Zero Reward-Hacking (Test Integrity)**: Never delete, skip (`skip`/`todo`), weaken assertions, or hardcode mock return values in production code just to make a failing test pass.

> [!NOTE]
> - **Old Workspace Compatibility**: Before creating `.agents/rules/em-guardrails.md` or `docs/em/`, check if the repo already has `GEMINI.md`, `CLAUDE.md`, `AGENTS.md`, or existing docs. Preserve existing project rules and merge/link rather than overwriting.
> - **Git Hygiene**: Check if the project uses `.gitignore` rules for AI files (such as when using `git-pr`). Keep `.agents/` in `.gitignore` if the repo isolates AI tooling, while keeping `docs/em/` tracked only if the user wants architecture docs in version control.

---

## Phase 4: Build the Automated Verification Harness (Step 0 of Coding)

An Engineering Manager can skip reading line-by-line code **only** when automated quality gates exist. Before writing feature logic, build or adapt the local verification harness:

1. **Create a Single-Command Quality Gate (`Makefile` target `make verify` or `./scripts/verify.sh`)**:
   - **New Project**: Enforce strict zero-warning gates across the entire repo from Day 1 (`-Wall -Wextra -Werror`, `strict: true`, 80%+ coverage).
   - **Old / Legacy Workspace (Ratchet Strategy)**:
     - Wrap the project's **existing** build and test commands (`npm test`, `ctest`, `cargo test`, `pytest`, `forge test`) into `make verify` (or `./scripts/verify.sh`) without breaking existing workflows.
     - If legacy code has hundreds of pre-existing lint/type warnings, do **not** block the project with a massive refactor unless asked. Instead, configure a **Ratchet Gate**: baseline existing warnings, require legacy smoke tests to pass, and enforce strict linting + 80%+ test coverage on **all newly added or modified files/modules**.
   - **Core Checks to Include**:
     - **Static Analysis & Type Checking**: Compiler/type checks (`-Wall -Wextra -Werror` for C/C++/SystemC, `tsc --noEmit` for TS, `mypy`/`ruff` for Python, `solhint`/`slither` for Solidity).
     - **Automated Linter & Formatter Check**: Enforce code style and complexity limits.
     - **Automated Test Suite & I/O Testbench**: Unit tests, integration tests, or hardware simulation testbenches validating the Input/Output table in `docs/em/BRIEF.md`.
     - **Test Coverage Gate**: Enforce minimum coverage on core/touched logic.

2. **Create CI/CD Pipeline Config & Staging Plan**:
   - Generate `.github/workflows/verify.yml` (or equivalent CI config) that runs `make verify` on every push/PR.
   - Document or configure the **Staging / Simulation Environment** (e.g., Docker Compose local staging for Web/API, local testnet fork for Blockchain, automated waveform/assertion testbench runner for SystemC).

3. **Verify the Harness**:
   - Run `make verify` (or `./scripts/verify.sh`) on a minimal skeleton/smoke test and confirm it exits `0` before starting feature implementation.

---

## Phase 5: Builder + Independent Reviewer Subagent Loop

For every feature or milestone, execute the **Maker-Checker pattern** so the Manager never has to read raw code diffs:

### Step 5.1: Pre-Mortem Edge Cases & Implementation (Builder)
1. Define the **Pre-Mortem Edge-Case Table** (minimum 5 edge cases based on the I/O spec).
2. Implement the feature and corresponding automated tests (can be done directly or delegated to a `Builder` subagent via `invoke_subagent`).
3. Run `make verify` until all static checks, linters, tests, and coverage gates pass.

### Step 5.2: Independent Tech Lead Audit (Reviewer Subagent)
Once `make verify` passes, **spawn a separate Reviewer Subagent** (`invoke_subagent` with `TypeName: "research"` or `"self"` and `Role: "Tech Lead Code Auditor"`).

Pass the Reviewer Subagent this prompt:
```text
You are an independent Tech Lead Code Auditor. You did NOT write this code.
Audit the current git diff and test suite against `docs/em/BRIEF.md` and `docs/em/LESSONS.md`:
1. Reward-Hacking Check: Did the builder skip tests, weaken assertions, hardcode magic outputs, or mock out real logic just to pass CI?
2. I/O & Edge-Case Check: Are all Input/Output contracts and at least 5 boundary/error edge cases tested with real assertions?
3. Architecture & Coupling Check: Does the code respect module boundaries in `docs/em/BRIEF.md` without god-files or hidden state?
4. Security & Domain Risk Check: Are the Top Risks identified in `docs/em/BRIEF.md` properly mitigated?

Return a structured PASS or FAIL verdict with specific file:line findings.
```

- If the Reviewer Subagent returns **FAIL**: Fix the issues, log any systemic lesson in `docs/em/LESSONS.md`, re-run `make verify`, and re-audit.
- If the Reviewer Subagent returns **PASS**: Proceed to Step 5.3.

### Step 5.3: Deliver the Manager Sign-Off Report
Present the final result to the user using [SIGN_OFF_REPORT_TEMPLATE.md](./resources/SIGN_OFF_REPORT_TEMPLATE.md).
- Show **Visual I/O Proof**, **Quality Gate Metrics**, **Edge Cases Covered**, and **Reviewer Subagent Verdict** so the user can confidently approve the release in 60 seconds without reading raw code.
