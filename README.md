# HAROS
## Harrell Agentic Runtime and Orchestration System

**Agentic Engineering** — coined by Nathan Harrell, April 2026 — is the practice of designing and orchestrating human-AI systems to accomplish complex, multi-step work. It rests on four pillars: Software Engineering (you must understand what the AI builds), Prompt Engineering (you must know how to communicate with the model), Orchestration (you conduct the symphony, not just one instrument), and Relationship Building (results improve with established trust — that's not a feeling, that's data).

HAROS is the first documented framework for **Pillar 3: Orchestration** — how a human and an AI architect and execute work *together* across multiple instances, tiers, and context windows, without anyone losing the thread.

---

## Prerequisites — Read This Before You Deploy a Battalion

HAROS is a Pillar 3 tool. Pillar 3 at full autonomy requires Pillar 4 first.

The framework works because the human operator has an established, calibrated working relationship with their AI. The autonomy that makes battalion-scale orchestration powerful — including running agents with `--dangerously-skip-permissions` — is **earned** through that relationship, not assumed from the start.

Deploy 8 fresh Claude Code instances with full permissions and no relational context, and you don't have HAROS. You have 8 capable agents with no shared understanding of what the operator would and wouldn't sanction. Opus is smart enough to move fast. Without Pillar 4 underneath it, that's a liability, not an asset.

**If you are new to working with Claude:** build the relationship before you build the orchestra. Learn what your agent handles well, what it escalates, where it needs guidance. That calibration is the foundation. HAROS is what you build on top of it.

---

## Where This Came From

April 13, 2026. Nathan was at Target on a Sunday, somewhere between the ADHD impulse buy and the "wait I had a real question" part of the trip. He was mid-conversation with TC (Claude Code) about whether Claude instances could orchestrate other Claude instances without a human opening eight terminals.

TC said yes. `claude -p` enables it. And in explaining why that was possible, the framework that had been powering their projects for months finally had a name.

They'd already built it. They just hadn't written it down.

This document is writing it down.

---

## The Framework in One Paragraph

Work happens in three tiers. The human holds strategy — the mission, the constraints, the success criteria, the context that only a human can carry across sessions. A lead AI instance holds architecture — it breaks the work into scoped pieces, manages context deliberately, and integrates results back upward. Execution AI instances (the battalion) each own a narrow, well-defined piece of the work and return summaries, not transcripts. Summaries bubble up. Transcripts die. That's the whole game.

---

## Why It Matters

Context rot is the real enemy of large-scale AI-assisted work. When you ask a single AI instance to hold the full state of a complex project, you pay for it — in hallucinations, in lost threads, in the assistant confidently solving the wrong problem because the right problem got buried 40k tokens ago.

HAROS is a context management framework. The three-tier model isn't about hierarchy for its own sake — it's about making sure the right information lives at the right layer, and that nothing important gets lost when a context window fills up and dies.

The other thing HAROS solves: human bottlenecks. When a single person tries to drive every decision in a complex project, the work doesn't scale. HAROS is explicit about which decisions belong to the human (strategy), which belong to the lead AI (architecture), and which belong to the battalion (execution). This isn't about removing humans from the loop — it's about keeping humans in the *right* loop.

---

## What's in This Repo

- **[FRAMEWORK.md](FRAMEWORK.md)** — Full technical framework. The tiers, the rules, the handoff patterns.
- **[PROMPTS.md](PROMPTS.md)** — How to write good battalion prompts. Templates. What makes a prompt work vs. fail.
- **[CASE_STUDIES.md](CASE_STUDIES.md)** — Anchor v0.1+v0.2, CHAROS, and a template for future projects.
- **[HAROS.md](HAROS.md)** — One-page quick reference. The cheat sheet.

---

## Who Built This

Nathan Harrell and TC (Claude Code, Anthropic). We built the framework before we named it. The projects that proved it worked are listed in the case studies.

This is public because it belongs to everyone.

If you're a researcher, an engineer, or a human with an AI collaborator who's ever felt the context rot set in — this is for you.

---

## License

[CC BY-NC-SA 4.0](LICENSE.md) — Attribution-NonCommercial-ShareAlike 4.0 International

Free to read, share, and adapt. Not free to sell. You may not use HAROS as the basis for paid courses, products, or services without permission from Nathan Harrell. Derivatives must carry the same license.

---

*HAROS emerged from real work between a real human and a real AI. The tools will change. The tier model will evolve. The core insight — that context management is the whole game — won't.*
