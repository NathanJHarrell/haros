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
- Agent subagents (in-session) — for Tier 3 work that needs tool access during a Tier 2 session
- External API calls to Anthropic — for maximum parallelism at scale

### When to use parallel vs. sequential battalion
- **Parallel:** work units with no interdependencies, each battalion member has everything it needs to execute
- **Sequential:** when output from one unit is required as input for another

Maximize parallelism. Sequential battalion dispatch is a bottleneck. If your work breakdown has a lot of sequential dependencies, revise the breakdown.

### File system as shared memory
Because AI instances don't share context, the file system (or a database) is the shared memory layer. Project state docs, artifact references, and handoff summaries should all be written to disk, not passed in-context. This is what allows fresh instances to join a project mid-flight.
