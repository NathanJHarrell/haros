# HAROS Quick Reference
## The Cheat Sheet

---

## Codenames (Pokémon Evolution Line)

| Tier | Role | Codename | Description |
|------|------|----------|-------------|
| 1 | Human | **Nathan** | Strategic layer. Defines mission, sets constraints, makes go/no-go calls. |
| 2 | Orchestrator | **Charizard** | TC. Fully evolved. Sees the whole battlefield. Spawns Charmeleons, writes prompts, monitors via FOV, delivers PRs. |
| 2 | Team Lead | **Charmeleon** | Headless Claude. Reviews prompts, decides if research needed, manages Charmanders, validates work, interfaces with Codex. |
| 3 | Worker | **Charmander** | Headless Claude/Sonnet/Haiku. Faithfully executes. Reports back. No questions. |
| 3 | Adversarial Review | **Codex** | Headless Codex (GPT-5.4). Independent review. Fresh eyes. Approval or denial with reasons. |
| 3 | Research Agent | **Perplexity** | MCP-powered deep web research with citations. Runs before dispatch when needed. |

---

## What Tier Am I In?

**Am I the human?**
→ You're **Tier 1 (Nathan)**. Define the mission. Set constraints. Review integration reports. Make strategic calls. You are not the bottleneck — stay at the level of decisions.

**Am I the lead AI instance, coordinating other instances?**
→ You're **Tier 2 (Charizard / Charmeleon)**. Break the work down. Write battalion prompts. Synthesize summaries. Manage your context budget. Do NOT fill your window with raw battalion output.

**Am I a spawned instance with a specific job to do?**
→ You're **Tier 3 (Charmander / Codex / Perplexity)**. Execute your scope. Return a summary — not a transcript. Surface blockers immediately. Don't make architectural decisions. Don't start work that wasn't in your prompt.

---

## What Do I Return?

| Tier | Returns |
|------|---------|
| Tier 3 → Tier 2 | Structured summary: status, artifact paths, brief design rationale, blockers |
| Tier 2 → Tier 1 | Integration report: what's done, what's in progress, what needs a human decision |
| Tier 1 → Tier 2 | Strategic direction: mission update, constraint change, or go/no-go on a recommendation |

**Never return a transcript.** Transcripts are context poison. Summarize.

---

## How Do I Hand Off?

### Tier 1 → Tier 2 (Kickoff)
Provide:
- Mission statement (specific, not vague)
- Hard constraints
- Success criteria
- Existing project state doc (if resuming)

Wait for the lead to restate the mission and ask clarifying questions before authorizing the work breakdown.

### Tier 2 → Tier 3 (Dispatch)
Write a battalion prompt with:
- Role statement
- Minimum necessary context
- Specific deliverable
- Hard constraints (scope fences)
- Return format

See PROMPTS.md for the full template.

### Tier 3 → Tier 2 (Return)
Return your summary in the format the prompt specified. If no format was specified, use:

```
Status: complete / partial / blocked
Artifacts: [file paths or references]
Summary: [2-3 sentences]
Blockers: [anything the lead needs to know]
```

---

## The Cardinal Rules

1. **Summaries bubble up. Transcripts die.**
2. **Scope before spawn.** Never dispatch without a clear deliverable.
3. **Human holds strategy.** Lead holds architecture. Battalion holds execution.
4. **Failure is local.** A battalion failure is a spec problem, not a project problem.
5. **The prompt IS the spec.** If the prompt is vague, the output will be vague.

---

## Context Budget (Tier 2)

| Allocation | Use |
|---|---|
| ~20% | Strategic brief + project state doc |
| ~40% | Active architecture work (writing prompts, evaluating summaries) |
| ~30% | Integrated battalion summaries |
| ~10% | Safety margin |

When budget nears full: write a project state handoff doc to disk, start a fresh Tier 2 instance with that doc as its opening brief.

---

## Is This Task Big Enough to Need HAROS?

| Situation | Framework? |
|---|---|
| Simple task, clear scope, 1-2 hours | No. Just do it. |
| Complex task, multiple workstreams, parallel work possible | Tier 2 + Tier 3 |
| Large project, human strategic direction needed | All three tiers |

The framework has overhead. Don't apply it where it isn't needed. Apply it where it is.

---

## Quick Checklist Before Spawning Battalion

- [ ] Is the deliverable specific and measurable?
- [ ] Does the instance have minimum necessary context (not the full project history)?
- [ ] Are the hard constraints explicit?
- [ ] Is the return format specified?
- [ ] Do I know what I'll do with the summary when it comes back?

If any box is unchecked, the prompt isn't ready.

---

## Research Agent: Perplexity MCP

Charmeleons decide if a research agent is needed before dispatching Charmanders.

### Setup
```bash
claude mcp add perplexity \
  --env PERPLEXITY_API_KEY="<key>" \
  -- npx -y @perplexity-ai/mcp-server
```

### Tools Available

| Tool | Model | Use Case |
|------|-------|----------|
| `perplexity_search` | — | Quick facts, current info, URLs |
| `perplexity_ask` | sonar-pro | Questions needing web context |
| `perplexity_research` | sonar-deep-research | Complex topics, comprehensive analysis with citations |
| `perplexity_reason` | sonar-reasoning-pro | Logical analysis, architecture decisions |

### When to Research

| Situation | Research? |
|-----------|----------|
| Familiar codebase, known patterns | Skip |
| External APIs or libraries | Yes — `perplexity_search` or `perplexity_ask` |
| New architectural pattern | Yes — `perplexity_research` |
| Security implications | Yes — `perplexity_research` + `perplexity_reason` |
| Prior art / best practices | Yes — `perplexity_research` |

Research output is written to a brief file. Charmanders read it alongside their HCSF dispatch doc.

---

## HAROS-FOV (Field of View)

Web-based terminal viewer for monitoring builds. `http://localhost:4200`

- Landing page: all active builds across all machines
- Build page: tiled xterm.js terminals (all sessions for a build)
- Full view: expand one terminal, interactive
- Cross-machine: proxied over Tailscale

### haros CLI

```bash
haros list                        # All active builds
haros status [build]              # Session detail
haros spawn <build> <label>       # Create a session
haros attach <build> [label]      # Attach to a session
haros tree <build>                # Agent hierarchy
haros cleanup [build] [--force]   # Kill stale sessions
haros kill <build> [--force]      # Tear down entire build
haros log <session> [lines]       # Capture output
haros broadcast <build> <message> # Send to all sessions
```

### Session Naming (updated)

```
haros-{build}-orch          → Charizard orchestrator
haros-{build}-lead{N}       → Charmeleon team lead
haros-{build}-worker{N}     → Charmander worker
haros-{build}-codex{N}      → Codex adversarial reviewer
haros-{build}-research      → Perplexity research agent
```
