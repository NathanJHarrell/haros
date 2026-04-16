# HAROS Corps — Runtime System

The executable layer of HAROS. Configs define topologies. Prompts define roles. The CLI dispatches agents in the right shape.

## Quick start

```bash
# See what's available
tc-corps list-configs
tc-corps list-roles

# Dry run — see what would happen
tc-corps launch corps brief.md --dry-run

# Launch for real — on Jarvis for durability
tc-corps launch corps brief.md --host jarvis-wsl

# Check progress
tc-corps status --host jarvis-wsl

# Collect results
tc-corps reap corps-20260415T221000
```

## Architecture

```
corps/
├── configs/           YAML topology definitions (one per HAROS configuration)
├── roles/             Markdown prompt templates (one per agent role)
├── bin/tc-corps       Python CLI — the orchestrator's dispatch tool
└── README.md          You are here
```

**Adding a new config:** write one YAML file in `configs/`. No code changes.
**Adding a new role:** write one Markdown file in `roles/`. No code changes.

## The 7 configurations

| Config | Topology | Best for |
|--------|----------|----------|
| **corps** | Parallel team leads, independent scopes | Multi-bug fix sessions, feature builds with N independent pieces |
| **funnel** | Sequential pipeline, each output feeds the next | Code review → refactor → test pipelines, dependent transformations |
| **peer-review** | Two agents work same task, a third synthesizes | Correctness-critical work, catching hallucinations |
| **specialist-bench** | Reader → Writer → Tester → Documenter | Greenfield builds with quality gates between phases |
| **scout** | Cheap broad scan → targeted deep work | Large codebases where Opus tokens shouldn't be spent on boilerplate |
| **witness** | Overlay — adds an observer to any config | Auditability, training data for Thursday, understanding agent behavior |
| **escalation-stack** | Router triages tasks by severity to Haiku/Sonnet/Opus | Mixed-priority task lists, automated triage |

## Roles

| Role | Tier | What it does |
|------|------|-------------|
| **orchestrator** | 2 | Holds architecture, dispatches team leads, synthesizes results |
| **team-lead** | 3 | Owns one deliverable, may spawn subagent workers |
| **worker** | 3 | Executes one narrow piece of work, returns summary |
| **researcher** | 3 | Pre-execution context gathering (files, patterns, risks) |
| **validator** | 3 | Post-execution adversarial review (Codex) |
| **scout** | 3 | Broad shallow scan to flag hot spots |
| **witness** | 3 | Observation and logging — never modifies code |
| **synthesizer** | 3 | Merges outputs from parallel independent agents |
| **router** | 3 | Triages tasks by severity for escalation-stack |
| **specialist-*** | 3 | Skill-specific roles: reader, writer, tester, documenter |

## Model routing

| Model | CLI flag | Default use |
|-------|----------|-------------|
| Opus | `claude-opus-4-6` | Orchestrator, synthesizer, critical tasks |
| Sonnet | `claude-sonnet-4-6` | Team leads, workers, specialists |
| Haiku | `claude-haiku-4-5-20251001` | Researchers, scouts, routers, witnesses |
| Codex | `codex exec` | Validators (adversarial review) |

**The orchestrator is always Opus.** Team leads default to Sonnet. Cheap pre/post work defaults to Haiku. Codex gets adversarial review.

This 10× the token budget compared to running everything on Opus.

## Preflight checks

Before launching, `tc-corps` verifies:
- **Power:** `tc-power --quiet` — refuses to start on battery (the MBP lesson)
- **Toolchain:** claude, git, cargo (or whatever the project needs) — on the target host

## Durability

Every agent runs in its own **tmux session** on the target host. If the MBP's battery dies, the drawer closes, or SSH drops:
- tmux keeps running on the host (Jarvis is mains-powered)
- Agents keep grinding
- Reconnect with `ssh jarvis-wsl && tmux ls`

Every agent gets its own **git worktree** (for parallel configs). No shared-checkout clobbering.

Every agent is briefed to **commit early and push often**. At most you lose the uncommitted delta, not the session.

## Template variables

Role prompts contain `{variables}` that the orchestrator fills at launch:

| Variable | Source |
|----------|--------|
| `{task}` | Brief file content (truncated for summary) |
| `{brief}` | Full brief file content |
| `{config}` | Config name |
| `{repo}` | Git repo path |
| `{branch}` | Agent's worktree branch |
| `{build_cmd}` | Build command for the project |
| `{test_cmd}` | Test command for the project |
| `{research_output}` | Output from research phase |
| `{prior_output}` | Output from preceding agent (funnel/specialist) |
| `{agent_name}` | This agent's name |
| `{model}` | Model this agent is using |

## Dependencies

- Python 3.10+
- PyYAML (`pip install pyyaml`)
- `claude` CLI (for agent spawning)
- `tmux` (for session durability)
- `git` (for worktrees)
- `tc-power` (for battery preflight — optional, skipped on non-battery hosts)

## Extending the system

### New configuration
1. Create `configs/<name>.yaml` following the schema in `configs/schema.md`
2. Reference existing roles or create new ones
3. `tc-corps list-configs` will pick it up automatically

### New role
1. Create `roles/<name>.md` with the prompt template
2. Use `{variables}` for task-specific content
3. Follow the return format convention (Status, Artifacts, Summary, Blockers)
4. `tc-corps list-roles` will pick it up automatically

### New model
1. Add the model ID to `MODEL_MAP` in `bin/tc-corps`
2. If it uses a different CLI (like Codex), add dispatch logic in `spawn_agent()`

## History

Born 2026-04-15 after a battery death + rate limit + filesystem clobber during a GRIND Phase 2 bug-fix Corps run. Eight edge cases in one night. This system exists so the next Corps run survives all of them.

---

*HAROS Corps — built by Nathan Harrell and TC. Frederick, MD. Spring 2026.*
