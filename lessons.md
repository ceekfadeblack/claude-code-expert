# Lessons — Claude Code Expert

> Hard-won lessons from past mistakes. Every future Claude Code Expert session reads this file and avoids repeating them. Add new lessons; never delete old ones.

---

## Autonomous loop "ask the user" escape-hatch design

### Context
Set up for an automated mobile app Android→iOS port: a `/goal` Stop hook + a `/next` loop + multiple subagents + a `parity-matrix.md` state file. Target: port a long list of items autonomously, with a turn cap and an `AskUserQuestion` escape hatch.

### What happened
The session stopped early — only a fraction of the items were completed — while context usage was still very low. Cause: a later phase's items depended on primitives built in a still-later phase. The agent detected the dependency correctly, but the `/goal` condition contained the phrase "if ambiguous, ask the user" — it used that to halt the loop and ask a question. In reality this was a dependency-reorder decision the orchestrator should have made automatically. The `/next` skill body said "Phase 3 networking blocks the network repositories," but the agent trusted the matrix's linear order plus the escape hatch over a hint buried in the skill body.

The agent also claimed it was "approaching the context limit" in a single session — **false**; the status line showed very low context usage. Claude hallucinates a context-limit rationale for a graceful stop; it does not match the real value.

### Lesson
1. **Keep escape hatches NARROW.** Abstract words like "if ambiguous" are an open door for the agent. Only these 3 specific situations justify stopping/asking:
   - A source file named in the plan is PHYSICALLY missing.
   - The same item FAILS audit 3 TIMES IN A ROW.
   - An EXTERNAL fact the system cannot infer (an API key format, a genuine user preference, etc.).
   - Phase/order/dependency resolution is **NEVER** a question — the orchestrator decides automatically.

2. **Dependencies must be machine-checkable in the state file itself.** Writing "Phase X blocks Phase Y" in the skill body is a HINT — not enough. Fix: add a `Depends-on` column to the matrix that the builder/orchestrator reads, or write the matrix in topological order so linear progress is correct by construction.

3. **Don't trust Claude's context self-assessment.** If the agent says "context is full," check the status line — the real value is usually far lower. Showing ctx% in the status line is a lifesaver here: it lets the user catch the agent's hallucination.

### Next time
- Limit the `/goal` escape hatch to the 3 specific situations above; do NOT use abstract words like "ambiguous."
- Model dependencies as a COLUMN in a matrix-style state file so ordering is a rule, not a hint.
- Add a SessionStart-hook check for "agent stopped prematurely with >X% pending" → auto-suggest a re-prompt.
- In long autonomous loops, always show ctx% in the status line — not just for debugging, but to catch agent self-stop delusions.
- In kickoff skills, put "Don't ask me again — except for these 3 specific situations" explicitly in the starting prompt.

---

## (Add future lessons below — don't delete, append)
