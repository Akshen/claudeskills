---
name: systematic-debugging
description: Applies a disciplined root-cause debugging methodology instead of guess-and-patch fixes. Use this skill whenever the user reports a bug, an error, unexpected behavior, a failing test, or asks "why isn't this working / why is this broken / help me debug this." Ensures the actual root cause is found and a regression test is written, rather than patching symptoms.
---

# Systematic Debugging

A methodology for finding root causes instead of patching symptoms. Trigger this mindset any time you're diagnosing unexpected behavior, not just when explicitly asked to "debug."

## The Loop

1. **Reproduce** — Get a minimal, reliable way to trigger the bug before touching any code. If you can't reproduce it, you can't confirm you fixed it. Ask for exact steps/input/environment if the user's report doesn't give you enough to reproduce.

2. **Isolate** — Narrow down *where* the bug lives before guessing *why*. Useful tactics:
   - Binary search the code path (comment out / bypass halves to narrow the failing region)
   - Add logging/print statements at decision points rather than guessing which branch runs
   - Check recent changes first (`git log`, `git blame`) if this is a regression — what changed since it last worked?
   - Reduce the reproduction case to the smallest input/steps that still trigger it

3. **Form a hypothesis** — State explicitly what you think is wrong and why, based on evidence gathered in step 2 — not a guess pulled from pattern-matching similar-looking bugs.

4. **Verify the hypothesis before fixing** — Confirm it directly (a log line, a debugger breakpoint, a quick assertion) rather than jumping straight to a fix and hoping. If the hypothesis is wrong, go back to step 2 with the new information.

5. **Fix the root cause, not the symptom** — Ask: "does this fix address why the bad state occurred, or just suppress the visible error?" A null check that hides a crash isn't a fix if the null shouldn't have been possible in the first place — trace back to where it was introduced.

6. **Write a regression test** — Before considering the bug closed, write a test that reproduces the original failure and confirms it now passes. This test should fail on the old code and pass on the fixed code.

7. **Check for the same bug elsewhere** — If the root cause is a pattern (e.g. a missing null check, a wrong assumption about input shape), grep for the same pattern elsewhere in the codebase rather than assuming this was the only instance.

## Anti-patterns to avoid

- **Guess-and-patch**: changing code speculatively without confirming the hypothesis first, then re-running to see if it "seems to work."
- **Shotgun debugging**: changing many things at once, making it unclear which change actually fixed it (or if it's fixed at all vs. coincidentally not reproducing).
- **Fixing the stack trace, not the bug**: adding a try/catch or null check purely to stop an error from surfacing, without understanding why the bad state occurred.
- **Skipping reproduction**: attempting a fix based only on reading code, without ever confirming the bug actually reproduces with your understanding of it.
- **No regression test**: closing the bug without a test means it can silently come back later.

## When you can't reproduce it

If the bug is intermittent, environment-specific, or otherwise hard to reproduce:
- Gather as much context as possible first (logs, stack traces, exact environment, timing) rather than guessing
- Look for patterns across occurrences (does it always involve certain input, load, timing, concurrency?)
- Consider whether it's a race condition, resource exhaustion, or external dependency issue — these often need different tactics (stress testing, timing logs) than a simple logic bug
- If truly stuck, say so explicitly and propose what additional logging/monitoring would help catch it next time, rather than shipping a speculative fix with false confidence
