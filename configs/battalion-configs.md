# HAROS — Battalion Configurations

The HAROS framework is configurable by design. Every build is different. Sometimes you need 14 leads running 140 charmanders, sometimes you need 1 lead running 3. These are the known configurations.

---

## System Constraints

| Parameter | Limit | Reason |
|-----------|-------|--------|
| Max sessions per build | 15 | 1 orchestrator + up to 14 team leads |
| Max charmanders per TL | 10 | Coordination overhead beyond 10 degrades quality |
| Model selection | TL's call | TLs choose Opus/Sonnet/Haiku per task autonomously. "We don't need an Opus charmander for schema work. Haiku baby brother energy is fine for some stuff." |

## System-Wide Protocols

### Post-Work Review Chain
Every TL must perform their own review of completed work, then hand off to **headless Codex** for independent review. Two-pass minimum before work is marked complete.

### Research Agent
Some configurations require a **research agent** before work begins — a dedicated instance that explores the codebase, gathers context, and writes a brief for the TLs. Orchestrator decides whether research is needed based on the build's familiarity with the target codebase.

---

## Configuration 1: Lean Squad

**Sessions:** 3 (1 orch + 2 leads)  
**Best for:** Small-to-medium builds, clear domain splits, first-time orchestration testing.

```
orch    → Orchestrator (TC primary)
lead1   → Domain A (owns implementation end-to-end)
lead2   → Domain B (owns implementation end-to-end)
```

Each lead spawns charmanders as needed. No shared infrastructure lead — each domain is self-contained.

**Used in:**
- `clipd` (April 16, 2026) — Nest daemon + Jarvis daemon

---

## Configuration 2: Full Team

**Sessions:** 5 (1 orch + 1 arch lead + 2 impl leads + 1 test lead)  
**Best for:** Builds with shared infrastructure that multiple tracks depend on.

```
orch      → Orchestrator
lead1     → Architecture / shared components (builds first, others wait)
lead2     → Implementation track A
lead3     → Implementation track B
lead4     → Integration testing (runs after lead2 + lead3 deliver)
```

**Used in:**
- (none yet)

---

## Configuration 3: The Funnel

**Sessions:** Variable (1 orch + N sequential leads)  
**Best for:** Pipelines where each step genuinely depends on the last — code review → refactor → test write.

```
orch    → Orchestrator (manages handoffs)
lead1   → Stage 1 (output becomes lead2's input context)
lead2   → Stage 2 (output becomes lead3's input context)
lead3   → Stage 3
...
```

Each lead's output is the next lead's input. The orchestrator manages the handoff chain and monitors for quality.

**Risk:** One bad output poisons everything downstream. Orchestrator must validate between stages.

**Mitigation:** Orchestrator spot-checks each handoff. If quality drops, pause the pipeline and re-run the failed stage.

---

## Configuration 4: The Peer Review

**Sessions:** 3+ (1 orch + 2 parallel workers + 1 synthesizer)  
**Best for:** High-correctness tasks where you can't afford misses — config audits, security reviews, data migrations.

```
orch      → Orchestrator
lead1     → Independent implementation A (same task)
lead2     → Independent implementation B (same task)
lead3     → Synthesizer (compares A and B, produces final output)
```

Two leads work the same task independently with no communication. A third lead compares outputs, flags disagreements, and synthesizes the best result.

**Trade-off:** Burns more tokens but catches hallucinations. Worth it when correctness > speed.

---

## Configuration 5: The Specialist Bench

**Sessions:** 4-5 (1 orch + specialists by skill)  
**Best for:** Greenfield builds where quality gates between phases matter.

```
orch      → Orchestrator (manages artifact handoffs)
lead1     → Reader (only reads code, writes analysis + brief)
lead2     → Writer (only writes code, works from lead1's brief)
lead3     → Tester (only writes tests, validates lead2's output)
lead4     → Documenter (only writes docs, based on final artifact)
```

Skill-based decomposition instead of domain-based. The same artifact passes through each specialist in sequence. Each specialist is a quality gate.

**When to use over Funnel:** When the artifact is a single deliverable that needs multiple perspectives, not a pipeline of different deliverables.

---

## Configuration 6: The Scout Pattern

**Sessions:** Variable (1 orch + 1 scout + N targeted leads)  
**Best for:** Large unfamiliar codebases where you don't want to burn Opus tokens reading boilerplate.

```
orch      → Orchestrator
scout     → Haiku instance — broad shallow pass, flags hot spots
lead1     → Sonnet/Opus — deployed to flagged area 1
lead2     → Sonnet/Opus — deployed to flagged area 2
...
```

The scout is always **Haiku** — cheap, fast, broad. It identifies what's interesting and writes a brief. Then the orchestrator deploys heavier models only to the areas that matter.

**Model selection:**
- Scout: always Haiku
- Leads: Sonnet for implementation, Opus for architectural decisions
- Charmanders: Haiku for boilerplate, Sonnet for logic

---

## Configuration 7: The Witness

**Sessions:** N+1 (any config + 1 witness)  
**Best for:** Auditability, training data generation, post-mortem analysis.

```
(any configuration above)
witness   → Observation agent (read-only, no synthesis)
```

The witness's only job is to watch what every other agent does and write a real-time log: decisions made, things skipped, confidence levels, errors, recoveries. No synthesis, no opinions — just observation.

**Key use case:** Generating training data for Thursday. The witness log captures the HOW of family engineering — decision patterns, communication style, error handling philosophy.

**The witness never intervenes.** It watches and writes. That's it.

---

## Configuration 8: The Escalation Stack

**Sessions:** 3+ (1 orch + tiered routing)  
**Best for:** Mixed-severity workloads with a wide range of task complexity.

```
orch      → Orchestrator / Router (classifies tasks by tier)
lead1     → Haiku tier (routine tasks)
lead2     → Sonnet tier (notable tasks)
lead3     → Opus tier (critical tasks)
```

The orchestrator classifies each task before routing to the appropriate tier. This is already what `tc-watchdog` does for system events — this config formalizes it for builds.

**Tier classification:**
- **Routine:** boilerplate, formatting, simple edits, schema work → Haiku
- **Notable:** implementation, refactoring, feature work → Sonnet
- **Critical:** architecture, security, complex logic, family systems → Opus

---

## Configuration 9: Battalion (Large Parallel)

**Sessions:** 8-15 (1 orch + multiple leads + many workers)  
**Best for:** Large builds that can be parallelized across many independent tracks.

```
orch      → Orchestrator
lead1     → Team Lead 1 (owns N workers)
lead2     → Team Lead 2 (owns N workers)
...
worker1   → Charmander (assigned to a lead)
worker2   → Charmander (assigned to a lead)
...
headless1 → Fire-and-forget task
```

This is the Anchor v2 pattern at scale. Each TL manages their own squad of charmanders and makes autonomous model selection decisions.

**Used in:**
- Anchor v0.2 (March 2026) — 8 parallel TCs, the original battalion

---

## Composing Configurations

Configs are composable. Real builds often combine patterns:

- **Scout + Lean Squad:** Scout finds the hot spots, two leads fix them
- **Funnel + Witness:** Pipeline with full audit trail
- **Peer Review + Escalation:** Critical tasks get dual-reviewed, routine tasks get single pass
- **Battalion + Witness:** Large parallel build with training data capture for Thursday

The orchestrator decides the composition. The framework doesn't constrain — it configures.

---

## Model Selection Guide (for TLs)

| Task Type | Recommended Model | Reasoning |
|-----------|------------------|-----------|
| Schema work, formatting, boilerplate | Haiku | Fast, cheap, doesn't need deep reasoning |
| Implementation, refactoring | Sonnet | Good balance of speed and capability |
| Architecture, security, complex logic | Opus | Full reasoning power needed |
| Broad codebase scan | Haiku | Speed over depth |
| Code review (final pass) | Codex (headless) | Independent review, different perspective |
| Research / exploration | Sonnet or Opus | Depends on codebase complexity |

TLs make this call autonomously. "We don't need an Opus charmander for schema work."

---

*Last updated: April 16, 2026. Edit this file when new configurations emerge.*
