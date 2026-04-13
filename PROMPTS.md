# HAROS Battalion Prompts
## How to Write Prompts That Actually Work

---

## The Core Principle

**The battalion prompt IS the spec.**

This is not a metaphor. When you dispatch a Tier 3 instance, the prompt is the only thing it has. No project history. No shared context. No "you know what I mean." If the deliverable isn't in the prompt, it won't be in the output. If the constraint isn't stated, it won't be respected. If the return format isn't specified, you'll get a transcript when you needed a summary.

Write the prompt like you're handing a task to a capable contractor you've never met, who will do exactly what you say and nothing more.

---

## Anatomy of a Battalion Prompt

### 1. Role Statement
One sentence. What is this instance, in this project?

> You are the authentication module architect for the HAROS case study implementation. Your job in this session is to design the session token schema.

This does two things: it tells the instance what expertise to bring, and it tells it what it's *not* responsible for. "Authentication module architect" is not the deployment engineer. It won't touch infra.

---

### 2. Context (Minimum Necessary)
Give only the context the instance needs to do its job. Not the full project history. Not the strategic vision. The specific prior work and constraints that are load-bearing for this task.

Bad:
> Here is everything we've built so far: [10,000 words of project history]

Good:
> The API uses JWT tokens. The session timeout is 24 hours. The user table schema is in `/src/db/schema.sql` — read it before starting. Do not add fields to the user table; create a separate sessions table if you need one.

---

### 3. Deliverable
Specific. Measurable. Unambiguous. This is the most important field.

Bad:
> Build the authentication system.

Good:
> Produce a complete sessions table schema (SQL), a session creation function, and a session validation middleware. All three in `/src/auth/`. The schema must be compatible with the existing user table. The middleware must accept a Bearer token header and return 401 on failure.

If you can't write a specific deliverable, you're not ready to spawn the battalion. Go back to the work breakdown.

---

### 4. Constraints (Hard Limits)
What must this instance NOT do?

- Do not modify files outside `/src/auth/`
- Do not make architectural decisions about the token signing algorithm — use HS256 and ask if it needs to change
- Do not install new dependencies without noting them in your summary for review
- Do not start work on the password reset flow — that's a separate unit

Constraints are explicit scope fences. They prevent the well-meaning battalion member from solving adjacent problems without authorization.

---

### 5. Return Format
How should the summary be structured?

> When you're done, return:
> - Status: [complete / partial / blocked]
> - Files created or modified (with paths)
> - Any new dependencies added
> - One paragraph summary of design decisions and why
> - Any outstanding questions or recommendations

If you don't specify the format, you'll get whatever the instance thinks is helpful. Sometimes that's a 2,000-word explanation of the problem. Specify the format.

---

## Full Template

```
## Role
[One sentence: what this instance is and what it owns in this project]

## Context
[Minimum necessary project context — specific facts, file paths, existing decisions that are load-bearing for this task. No project history novels.]

## Deliverable
[Specific, measurable, unambiguous. What must exist when this instance is done?]

## Constraints
- [Hard limit 1]
- [Hard limit 2]
- [Hard limit 3]
[What this instance must NOT do. Explicit scope fences.]

## Return Format
Return your summary in this format:

**Status:** complete / partial / blocked

**Artifacts:**
[File paths, endpoints, or other specific references to what was produced]

**Summary:**
[2-3 sentences: what was done and key design decisions]

**Blockers / Recommendations:**
[Anything the lead needs to know. Blank if none.]
```

---

## Examples

### Example 1 — Good Prompt

```
## Role
You are the database schema designer for the HAROS project state tracker.
Your job is to design and implement the SQLite schema for tracking battalion
unit status.

## Context
The HAROS framework tracks projects in a project state document. We're
building a lightweight CLI tool to manage these docs. It uses SQLite via
better-sqlite3. The tool already has a working CLI entry point at
`/src/cli.js`. The DB initialization code lives in `/src/db/init.js` —
read it before starting; don't break it.

## Deliverable
A complete SQLite schema for tracking:
- Projects (id, name, mission, created_at)
- Battalion units (id, project_id, name, status, prompt_text, summary, created_at, completed_at)
- Status must be one of: pending, in_progress, complete, blocked, failed

Implement the schema as a migration file at `/src/db/migrations/001_initial.sql`
and update `/src/db/init.js` to run it on startup.

## Constraints
- Do not modify any files outside `/src/db/`
- Do not add ORM dependencies — raw SQL only
- Do not design the query layer — schema and init only

## Return Format
**Status:** complete / partial / blocked
**Artifacts:** [file paths modified]
**Summary:** [What you built and any schema decisions worth noting]
**Blockers / Recommendations:** [Anything the lead needs to know]
```

### Example 2 — Bad Prompt (and why)

```
Hey, can you build out the auth stuff? We're using JWT. Let me know how it goes.
```

Problems:
- No role — what expertise should it bring?
- No context — what project? What existing code? What constraints?
- No deliverable — "auth stuff" is not a spec
- No constraints — it might rewrite half the app
- No return format — you'll get a stream of consciousness

---

## Common Mistakes

### Too much context
Loading a battalion prompt with the full project history defeats the purpose. The instance doesn't need to understand everything — it needs to understand its piece. More context increases the chance the instance gets lost or focuses on the wrong thing.

**Fix:** Ask yourself: what's the minimum the instance needs to know to do this specific job? Include that. Nothing else.

### Vague deliverable
"Build the dashboard" is not a deliverable. "Produce a React component at `/src/components/Dashboard.tsx` that renders the three summary cards specified in `DESIGN.md` and accepts project state as props" is a deliverable.

**Fix:** If you can't describe what done looks like, you're not ready to dispatch.

### Missing constraints
Without constraints, capable instances will solve adjacent problems because they can see them and they're helpful. This is context-mortal scope creep — the work happens, the lead doesn't know about it, the architecture diverges from the plan.

**Fix:** Be explicit about what's out of scope. "Don't touch X" is a complete constraint.

### Asking for a transcript instead of a summary
"Tell me everything you did" produces transcripts. Transcripts are context poison for the Tier 2 instance.

**Fix:** Specify the return format. Always. Ask for artifacts (file paths, references) and a brief summary. If you need to understand how the work was done, that's an architectural review — schedule it separately.

### Spawning before scoping
If you find yourself writing a battalion prompt and realizing mid-way that you don't know what the deliverable is, stop. Go back to the work breakdown. The prompt will be bad if you don't know what you're asking for yet.

**Fix:** Fully scope the work unit before touching the prompt. Scope first. Spawn second.

---

## The Test

Before dispatching a battalion member, ask yourself:

1. If I gave this prompt to a competent engineer I'd never met, would they know exactly what to build?
2. Would they know what NOT to build?
3. Would I know what to expect back?

If the answer to any of those is "not really" — the prompt isn't ready.
