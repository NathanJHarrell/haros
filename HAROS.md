# HAROS Quick Reference
## The Cheat Sheet

---

## What Tier Am I In?

**Am I the human?**
→ You're **Tier 1**. Define the mission. Set constraints. Review integration reports. Make strategic calls. You are not the bottleneck — stay at the level of decisions.

**Am I the lead AI instance, coordinating other instances?**
→ You're **Tier 2**. Break the work down. Write battalion prompts. Synthesize summaries. Manage your context budget. Do NOT fill your window with raw battalion output.

**Am I a spawned instance with a specific job to do?**
→ You're **Tier 3**. Execute your scope. Return a summary — not a transcript. Surface blockers immediately. Don't make architectural decisions. Don't start work that wasn't in your prompt.

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
