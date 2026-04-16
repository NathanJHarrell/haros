# HAROS Configuration Schema

A configuration defines a **topology** — who talks to whom, in what order, using which models.

## Config file format (YAML)

```yaml
name: string           # kebab-case identifier, matches filename
description: string    # one-line human-readable summary

# Phases execute in order. Each phase completes before the next begins.
phases:
  - name: string       # phase identifier
    parallel: bool     # true = all roles in this phase run concurrently
                       # false (default) = roles run sequentially in list order
    optional: bool     # true = orchestrator may skip this phase based on task
    roles:
      - role: string   # references a file in roles/ (e.g. "team-lead" → roles/team-lead.md)
        model: string  # model to use: opus | sonnet | haiku | codex
        count: int | "{variable}"  # how many instances of this role to spawn
                                   # use "{variable}" for orchestrator-decided counts
        receives_from: string      # phase.role whose output feeds this role's context
                                   # omit if role receives from orchestrator directly
        spawns_workers: bool       # if true, this role may use the Agent tool to spawn sub-agents

# Overlays modify any config by adding cross-cutting agents
overlays:
  - name: string       # e.g. "witness"
    role: string       # role template to use
    model: string      # model assignment
    observes: string   # "all" or specific phase name

# Defaults applied to all agents unless overridden per-role
defaults:
  worktree: bool       # create git worktree per agent (default: true)
  tmux: bool           # run each agent in its own tmux session (default: true)
  commit_interval: string  # auto-commit frequency, e.g. "10m" (default: "10m")
  host: string         # where to run: "local" | "jarvis-wsl" (default: "local")
  preflight:           # checks that must pass before launch
    - power            # tc-power --quiet
    - toolchain        # cargo, claude, git, etc.
```

## Conventions

- Config files live in `corps/configs/<name>.yaml`
- Role templates live in `corps/roles/<role>.md`
- Configs reference roles by name; the CLI resolves `roles/<name>.md`
- The orchestrator (Tier 2 TC) reads the config and populates role prompts with task-specific context using `{variable}` substitution
- Codex is treated as a model, not a role — `model: codex` routes to `codex exec` instead of `claude`
- Adding a new config = one YAML file. Adding a new role = one Markdown file. No code changes required.

## Template variables

Role prompts may contain `{variables}` that the orchestrator fills at launch time:

| Variable | Source |
|----------|--------|
| `{task}` | The task description from the launch command |
| `{repo}` | The git repo path |
| `{branch}` | The git branch name for this agent's worktree |
| `{brief}` | Contents of the brief file passed at launch |
| `{research_output}` | Output from the research phase (if any) |
| `{prior_output}` | Output from the preceding agent (funnel configs) |
| `{agent_name}` | This agent's assigned name (e.g. "tl-scrollback") |
| `{model}` | The model this agent is running on |
