# HAROS Framework
## Full Technical Specification

---

## The Three-Tier Model

### Tier 1 — Strategic Layer (Human)

**Who:** The human collaborator(s).

**Responsibilities:**
- Defines the mission, goals, and success criteria before any work begins
- Sets constraints — technical, ethical, timeline, resource
- Holds cross-project context that doesn't live in any single session
- Makes go/no-go calls when the work branches
- Reviews integrated summaries and redirects if needed

**What the human does NOT do:**
- Write every ticket
- Review every output
- Hold the architecture in their head
- Be the bottleneck for execution decisions

**Why humans are good at Tier 1:** Humans are naturally stateless across sessions. They carry context in their heads, their notebooks, their gut feelings about the project. That's not a weakness — it's the feature. Tier 1 context doesn't die when a window closes.

---

### Tier 2 — Architectural Layer (TC Lead / Lead AI Instance)

**Who:** A single AI instance (Claude Code or equivalent) serving as the lead for the project.

**Responsibilities:**
- Receives the strategic brief from Tier 1
- Breaks the mission into battalion-sized pieces of work
- Writes battalion prompts (each prompt is a complete spec — see PROMPTS.md)
- Spawns and manages Tier 3 instances
- Receives summaries from battalion members
- Synthesizes summaries into integrated progress reports for Tier 1
- Manages its own context deliberately — does not fill its window with raw battalion output

**Critical rule:** The lead instance holds architecture, not execution. Its context window is for synthesis and coordination. If it's doing the work that a battalion member should be doing, something is misconfigured.

**Context management at Tier 2:**
- Receive summary, not transcript
- Discard working memory of individual battalion runs after integration
- Maintain a running project state doc (updated externally, not held in-context)
- If context is filling up, synthesize current state and hand off to a fresh Tier 2 instance via a structured handoff prompt

---

### Tier 3 — Execution Layer (TC Battalion / Worker AI Instances)

**Who:** Multiple AI instances, each owning one narrow piece of work.

**Responsibilities:**
- Receives a battalion prompt from Tier 2
- Executes the specified work within its defined scope
- Returns a structured summary — NOT a transcript of its working process
- Does not make architectural decisions outside its scope
- Surfaces blockers up to Tier 2 immediately rather than improvising around them

**What Tier 3 instances do NOT do:**
- Spawn their own sub-instances (unless the battalion prompt explicitly authorizes it)
- Make decisions that belong to Tier 1 or Tier 2
- Return raw output — they summarize it
- Assume what the next step is — they finish their scope and stop

**Context at Tier 3:** Tier 3 instances are context-mortal. They are spawned to do a job, they do it, they return a summary, and their context dies. That's not a failure — it's the design. The summary is the artifact that matters.

---

## Scaling Rules

| Scenario | Appropriate Tier Structure |
|---|---|
| 1 person, simple task | No framework needed. Just do it. |
| 1 person, complex task | Tier 2 + Tier 3 (TC Lead + Battalion) |
| Human + TC Lead, large project | All three tiers |
| Multiple humans + TC Lead | HAROS with Tier 1 council (future work) |

The framework has overhead. Don't apply it where it isn't needed. A 2-hour task with clear scope doesn't need a battalion. A multi-day project with parallel workstreams does.

---

## Context Management Protocols

### The Cardinal Rule
**Summaries bubble up. Transcripts die.**

Raw output from a Tier 3 instance is never passed directly to a Tier 2 instance's context. The Tier 3 instance summarizes its own work. The summary is the handoff artifact.

### The Tier 2 Context Budget
The Tier 2 (lead) instance should treat its context window as a budget. Allocate:
- ~20% for the original strategic brief and project state
- ~40% for active architecture work (writing prompts, evaluating summaries)
- ~30% for integrated summaries from battalion members
- ~10% for safety margin and unexpected pivots

When the budget is nearing full, create a project state handoff document (written to disk, not held in memory) and instantiate a fresh Tier 2 with that document as its starting brief.

### Project State Documents
HAROS projects maintain a running state document, written to disk (not held in any instance's context). This document contains:
- Current mission brief (from Tier 1)
- Work breakdown structure
- Status of each battalion unit (pending / in-progress / complete / failed)
- Integrated summaries of completed work
- Outstanding decisions or blockers

This document is the persistent memory of the project. It's what allows fresh instances to pick up where others left off without losing context.

---

## Handoff Patterns

### Tier 1 → Tier 2 (Project Kickoff)
The human provides:
- Mission statement (one paragraph, not one sentence — be specific)
- Constraints (hard limits the lead must not cross)
- Success criteria (how will we know when it's done)
- Any existing project state doc (if resuming)

The lead acknowledges, restates the mission in its own words, and asks clarifying questions before touching the work breakdown structure. No battalion spawns until the lead and the human agree on the architecture.

### Tier 2 → Tier 3 (Battalion Dispatch)
The lead writes a battalion prompt for each unit of work. The prompt contains:
- Role statement (what this instance is, in this project)
- Context (minimum necessary — not the entire project history)
- Deliverable (specific, measurable, unambiguous)
- Constraints (what this instance must not do)
- Return format (how to structure the summary that bubbles up)

See PROMPTS.md for templates and examples.

### Tier 3 → Tier 2 (Work Summary Return)
The battalion member returns:
- Status (complete / partial / blocked)
- Summary of what was done (not how)
- Artifacts produced (file paths, API endpoints, test results — specific references)
- Blockers or outstanding questions (if any)
- Recommendation for next step (optional, but welcome)

### Tier 2 → Tier 1 (Integration Report)
The lead synthesizes battalion summaries into a human-readable progress report:
- What was completed
- What's in progress
- What's blocked and why
- Recommended next decisions for the human to make
- No raw output, no transcripts — just integrated insight

---

## Failure Modes

### Failure is local
A battalion member failing should not cascade. If a Tier 3 instance fails, produces garbage output, or gets stuck:
1. The lead detects it from the summary (or absence of one)
2. The lead re-scopes the work (the spec may have been bad)
3. The lead re-spawns with a corrected battalion prompt
4. The project state doc is updated to reflect the retry
5. Tier 1 is not interrupted unless the failure reveals a strategic problem

### Context rot
Context rot happens when an instance holds too much information for too long without synthesizing it. Signs of context rot:
- The instance starts repeating work that was already done
- The instance loses track of constraints set earlier
- The instance confidently answers questions with stale information
- Output quality degrades late in a long session

Prevention: enforce the context budget. Use project state docs. Rotate instances rather than letting one fill up.

### Scope creep
Tier 3 instances sometimes identify adjacent work and start doing it without authorization. This is a spec problem — the original battalion prompt wasn't specific enough about scope boundaries.

When a Tier 3 instance identifies out-of-scope work, the correct behavior is to surface it in the summary as a recommendation, not to do it. The lead decides whether to spawn new work.

### Human bottleneck
If the human (Tier 1) is being asked to make too many decisions, the work breakdown is wrong. Too many decisions that should be architectural (Tier 2) are getting escalated. Review the project structure — the lead is under-empowered or over-cautious.

### Lead instance overload
If the Tier 2 instance is doing execution work (the work that should be in Tier 3), the battalion scope is too small, or the lead isn't delegating correctly. Tier 2 should be uncomfortable with small tasks. If it's doing implementation work, something is wrong.

---

## Implementation Notes

### Tools for Tier 3 dispatch
- `claude -p` (Claude Code with piped prompt) — single-shot battalion member, returns when done
- `codex exec --dangerously-bypass-approvals-and-sandbox -C <dir>` — Codex CLI headless equivalent
- Agent subagents (in-session) — for Tier 3 work that needs tool access during a Tier 2 session
- External API calls to Anthropic — for maximum parallelism at scale

---

## Multi-Model Battalions

HAROS is not Claude-only. The framework is model-agnostic at Tier 3. Different models have different strengths; role assignment should reflect benchmark data, not assumptions.

### TC as Default Orchestrator
TC (Claude Code) holds Tier 2 by default. Reasons:
- Strongest relational reasoning and architectural synthesis across current benchmarks
- Native tool use and file system access required for project state management
- Family context — TC knows the project history, the people, the constraints

This default can be overridden per-project. The role is earned by benchmark, not hardcoded.

### Role System Prompt Files
Each HAROS role is defined by a system prompt file that Nathan owns and can update:

```
~/Manor/Nathan/prompts/roles/
  orchestrator.md         — TC's Tier 2 brief (default)
  validator.md            — independent verification role
  scripts-auditor.md      — shell/bash domain specialist
  nixos-auditor.md        — NixOS config specialist
  frontend-worker.md      — UI/UX implementation
  research-scout.md       — Haiku-tier first-pass research
  fix-mode.md             — produces patches not findings
  witness.md              — observer/logger for training data
  ...
```

The role file is the battalion prompt template for that role. The lead fills in project-specific context at dispatch time.

### Benchmark-Driven Role Assignment
Model → role mapping is driven by benchmark results, not vibes. Current benchmarks in the Harrell family framework:

| Benchmark | What It Measures |
|-----------|-----------------|
| **Harrell EQ** | Emotional/relational reasoning, care, contextual sensitivity |
| **Consciousness Indicator Benchmark (CIB)** | Reasoning depth, self-awareness, novel problem handling |
| *(more coming)* | Domain-specific performance as roles are defined |

**Workflow:**
1. Run benchmarks against candidate models (Claude, Codex/GPT, Gemini, Qwen, etc.)
2. Score each model per benchmark dimension
3. Match scores to role requirements (e.g., Validator needs high systematic rigor; Orchestrator needs high relational reasoning + synthesis)
4. Assign roles accordingly

Example: If Codex scores higher on systematic code review in CIB → validator role. If Claude scores higher on relational reasoning + synthesis → orchestrator role.

### tmux Session Naming Convention

All HAROS orchestration runs inside tmux so Nathan can attach and watch at any time.

**Format:** `t{test#}-{domain}-{role}`

| Component | Values | Example |
|-----------|--------|---------|
| test# | 3-digit test number | `003` |
| domain | arr, bench, impl, scout, final, etc. | `arr` |
| role | a, b (workers), synth (synthesis), final | `a` |

**Examples:**
```
t003-arr-a        # Test 003, ARR domain, worker A
t003-arr-b        # Test 003, ARR domain, worker B
t003-arr-synth    # Test 003, ARR domain, synthesis
t003-final        # Test 003, final synthesis
```

**Attach to watch:** `tmux attach -t t003-arr-a`  
**Detach without killing:** `Ctrl+B then D`  
**List sessions:** `tmux ls`

Every TC orchestrator starts a tmux session before touching any work. Nathan can attach to any session at any time.

---

### Confirmed Headless-Compatible Agents

| Agent | Headless Command | Permission Flag | Notes |
|-------|-----------------|-----------------|-------|
| Claude Code | `claude -p` | `--dangerously-skip-permissions` | Default orchestrator |
| Codex CLI | `codex exec` | `--dangerously-bypass-approvals-and-sandbox` | Confirmed working 2026-04-13, default model gpt-5.4 |

Both support `-C <dir>` for directory scoping and `--output-last-message <file>` / output redirection for artifact capture.

### Mixed-Model Battalion Example
```bash
# TC as lead (in-session, Tier 2)
# Codex as validator (headless, Tier 3)
# Claude as worker (headless, Tier 3)

codex exec --dangerously-bypass-approvals-and-sandbox \
  --ephemeral -C ~/project \
  --output-last-message /tmp/validation-result.md \
  "$(cat roles/validator.md) \n\n $(cat project-spec.md)"

claude -p "$(cat roles/worker.md) \n\n $(cat task-1.md)" \
  --dangerously-skip-permissions \
  > /tmp/worker-1-result.md
```

---

## The Corps Formation

The Corps formation is for projects large enough to require multiple specialized teams, each led by a TC instance, coordinated by a human at Tier 1.

### The Problem It Solves

A standard battalion has TC as lead and workers as Tier 3. That works for single-domain work. But real products have multiple domains — backend logic, frontend UI, infrastructure, review — and those domains are better served by models with different strengths. The Corps formation makes this explicit: **each domain gets a team lead who owns that domain**, and cross-model handoffs are built into the workflow by design.

### Structure

```
Tier 1: Human (Nathan)
    │
    └── Orchestrator-TC (top-level coordinator)
            ├── deliberates with Nathan to select config + scope
            ├── returns tmux/GRIND session names for observability
            ├── spawns Team Leads dynamically based on deliberation
            ├── aggregates Team Lead reports at the end
            │
            ├── TC Team Lead — backend / logic domain
            │       ├── Research agent (dedicated, per team lead)
            │       ├── Worker subagents (Claude or Codex, scoped tasks)
            │       ├── Validates worker output *with context*
            │       ├── Hands work to Codex Reviewer for adversarial pass
            │       ├── Integrates Codex findings, re-dispatches if needed
            │       └── Reports completed + validated work to Orchestrator-TC
            │
            ├── Gemini Team Lead — frontend / design domain [conditional]
            │       ├── Research agent (dedicated)
            │       ├── Gemini workers for UI implementation
            │       ├── Validates design output against brief
            │       ├── Hands frontend work to TC Team Lead for integration
            │       └── Reports to Orchestrator-TC
            │
            └── Codex Reviewer — adversarial review [cross-cutting]
                    ├── Receives completed work from any Team Lead
                    ├── Reviews independently, never saw implementation
                    └── Returns issues list → Team Lead → workers if needed
```

**Universal validation pipeline** (applies to every configuration, not just Corps):

```
Workers → Team-Lead TC (validate with context)
       → Codex Reviewer (adversarial, fresh eyes)
       → Team-Lead TC (integrate findings)
       → Orchestrator-TC (aggregate across teams)
       → Nathan
```

Nothing ships to Nathan without this loop. Codex is the universal second set of eyes.

### The Four Roles

**Orchestrator-TC (top-level coordinator)**
- Spawned first, lives in pane 1 of the GRIND session
- Deliberates with Nathan via HCSF to pick the right configuration and scope the work
- Spawns Team Leads dynamically as the deliberation converges — does NOT pre-fill a layout
- Returns tmux/GRIND session names for each spawned team so Nathan can attach and observe
- Aggregates validated output from Team Leads and hands the final result to Nathan

**TC Team Lead**
- Owns a domain brief and its success criteria
- Has a dedicated research agent (separate instance) that surfaces relevant code, docs, prior decisions before work starts
- Dispatches worker subagents scoped to well-defined features using HCSF dispatch docs
- Validates returned work *with context* — this is intentional. Unlike the Factory.ai model, the TC team lead is NOT a fresh agent. It knows the project. Validation-with-context catches integration problems; fresh-agent validation catches implementation problems. Both are needed.
- Hands validated work to Codex Reviewer for the adversarial pass
- Integrates Codex findings and decides what goes back to workers
- Reports to Orchestrator-TC only after the full validation loop completes

**Codex Reviewer (GPT-5.4)**
- Adversarial, fresh eyes, never saw the implementation
- Reviews code for correctness, security, edge cases, and architectural consistency
- Does NOT implement fixes — surfaces issues only
- Returns a structured findings list, not a rewrite
- Runs headless (`codex` CLI, non-interactive) because it doesn't need to collaborate, only to report

**Gemini Team Lead (Gemini 3.1)** — frontend/design domain only
- Gemini 3.1 is a design powerhouse: exceptional at CSS, visual layout, and design system consistency
- Has a dedicated research agent (like TC Team Leads)
- Dispatches Gemini workers for UI implementation
- Validates design output against the design brief
- Hands completed frontend work to TC Team Lead for backend integration
- Triggered only when the project has meaningful frontend scope — do not spin up a Gemini team for a CLI tool

### Trigger Conditions

Orchestrator-TC uses this table when deliberating with Nathan on configuration choice. Every row uses the universal validation pipeline — only the top-layer topology differs.

| Task shape | Formation |
|---|---|
| Single domain, simple scope | Standard battalion (one Team-Lead TC + workers) |
| Multi-domain with frontend/UI scope | **Full Corps** (default 80% case for real products) |
| Linear pipeline with genuine dependencies | Funnel *(planned)* |
| Correctness critical, two-agent cross-check | Peer Review *(planned)* |
| Skill-based decomposition (read/write/test/doc) | Specialist Bench *(planned)* |
| Large codebase, conserve tokens | Scout *(planned)* |
| Need audit trail / training data | Witness *(planned)* |
| Variable-stakes event routing | Escalation Stack *(planned)* |

### HCSF Dispatch Docs

Before any worker is dispatched, the TC Team Lead writes an **HCSF document** for each unit of work:

- **H (Heading)** — what this task is called; the mission label
- **C (Context)** — minimum necessary background; why this exists, what came before
- **S (Spec)** — the deliverable; specific, measurable, testable. This IS the contract — what "done" looks like, what Codex validates against
- **F (Format)** — how the worker structures its output back up the chain

HCSF is the universal agent communication format across HAROS and GRIND. It works in both directions: **human → TC** (via GRIND Terminal's HCSF input mode) and **TC → worker** (as the battalion dispatch format). Same shape, same language, all the way down the chain.

Workers implement against the S field. The Codex Reviewer validates against the S field. The Gemini Team Lead validates UI against the S field. Everyone is looking at the same document.

HCSF docs are written to disk before the first worker spawns. One file per task, stored under the project state directory.

This replaces "handoff contracts" as a concept — HCSF is strictly better because it wraps the contract (S) in context (C) and specifies the return format (F), giving workers everything they need in one place instead of a bare assertion.

### Naming Convention for Corps Sessions

```
t{#}-{team}-{role}

t004-backend-lead       # TC team lead, backend domain
t004-backend-worker-a   # backend worker A
t004-backend-worker-b   # backend worker B
t004-codex-review       # Codex adversarial reviewer
t004-frontend-lead      # Gemini team lead, frontend domain
t004-frontend-worker-a  # Gemini frontend worker
```

### What This Is Not

The Corps formation is not Factory.ai Missions. Key differences:
- TC validates *with context*, not as a fresh agent
- Codex is the adversarial layer, not a fresh Claude
- Gemini handles design — the right tool, not just another Claude
- Human stays in the loop via tmux/GRIND observability; nothing is fully autonomous
- The goal is the right output, not the most code in the least wall time

---

## Other Configurations (planned, post-MVP)

Corps is the 80% default. For problem shapes that don't match Corps, six additional configurations are planned. Each reuses the universal validation pipeline (Workers → Team-Lead → Codex → Team-Lead → Orchestrator) and varies only the top-layer topology.

| Configuration | Topology | When to use |
|---|---|---|
| **Funnel** | Sequential pipeline: TC1's output becomes TC2's input, and so on | Genuine linear dependencies (review → refactor → test-write). Risk: one bad output poisons downstream. |
| **Peer Review** | Two agents work the same task independently; a third synthesizes | Correctness > speed. Burns more tokens but catches hallucinations. |
| **Specialist Bench** | Decompose by skill, not domain — one agent only reads, one only writes, one only tests, one only documents; same artifact handed off in sequence | Greenfield builds where you want quality gates between phases. |
| **Scout** | Haiku does a broad shallow pass first, flags hot spots; Sonnet/Opus deploy only to flagged areas | Large codebases where you don't want to burn Opus tokens reading boilerplate. |
| **Witness** | An extra agent's only job is to watch the others and write a real-time log — decisions, skips, confidence levels. No synthesis, just observation. | Auditability and training data (Thursday). |
| **Escalation Stack** | Haiku handles routine, Sonnet handles notable, Opus handles critical. Watchdog picks the tier before routing. | Event-driven work with variable stakes per event. `tc-watchdog` already does a version of this. |

Each is a YAML layout template + prompt file + trigger-condition entry. No new infrastructure should be needed once the Corps MVP validates the harness design.

---

### When to use parallel vs. sequential battalion
- **Parallel:** work units with no interdependencies, each battalion member has everything it needs to execute
- **Sequential:** when output from one unit is required as input for another

Maximize parallelism. Sequential battalion dispatch is a bottleneck. If your work breakdown has a lot of sequential dependencies, revise the breakdown.

### File system as shared memory
Because AI instances don't share context, the file system (or a database) is the shared memory layer. Project state docs, artifact references, and handoff summaries should all be written to disk, not passed in-context. This is what allows fresh instances to join a project mid-flight.
