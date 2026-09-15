---
name: refactoring-safely
description: Governs how to restructure existing code without changing its behavior — small verifiable steps, tests green between each step, no bundling of refactoring with feature changes or bug fixes. Use this skill whenever the user asks to "refactor," "clean up," "simplify," "reorganize," or "extract" code, or when a fix/feature requires restructuring code first before the actual change. Distinct from code-review (evaluating code) and robust-coding-practices (writing new code) — this is specifically about changing structure while preserving behavior.
---

# Refactoring Safely

A discipline for restructuring code without breaking it. The defining rule: **refactoring changes structure, not behavior** — if behavior needs to change too, that's a separate step, not the same commit.

## Before starting

- Confirm there are tests covering the current behavior of the code being refactored. If there aren't, write characterization tests first (tests that capture *current* behavior, even if that behavior has quirks) — refactoring without a safety net is just rewriting with extra risk.
- Get the current test suite green before making any change, so you have a trustworthy baseline.
- Decide the target structure before starting, but expect to adjust as you go — don't try to plan every micro-step in advance.

## The discipline

1. **One small step at a time.** Extract one method, rename one variable, inline one function — not a batch of unrelated restructurings in one pass.
2. **Run tests after every step.** Not after the whole refactor — after each individual change. If tests fail, you know exactly which step broke it.
3. **Commit after each green step** (or at least a clear checkpoint you can revert to). Small commits make it trivial to bisect if something surfaces later.
4. **Never mix refactoring with behavior change.** If you notice a bug while refactoring, note it and fix it as a separate, subsequent step with its own test — don't fix it "while you're in there." Mixing the two makes it impossible to tell, if something breaks, whether it was the restructuring or the behavior change.
5. **Keep the public interface stable unless the refactor's explicit goal is to change it.** Internal restructuring shouldn't ripple into callers unless that's the point.

## Common safe refactoring moves

- **Extract function/method** — pull a block into a named function; verify tests pass; the extracted function should be independently testable if it has real logic.
- **Rename** — for clarity; use IDE/tool-assisted rename where possible to catch all references, not manual find-replace which risks partial matches.
- **Inline** — the reverse of extract, when a function/variable adds indirection without adding clarity.
- **Replace conditional with polymorphism / lookup table** — when a switch/if-chain keeps growing on the same type check.
- **Introduce parameter object** — when a function's parameter list has grown unwieldy.
- **Move function/class** — relocating to a more appropriate module; verify all imports/references update.

## Red flags that mean "stop and reassess"

- Tests fail after a step and it's not obvious why — don't keep making further changes on top; revert to the last green checkpoint and take a smaller step.
- You find yourself wanting to change three unrelated things at once — split into separate refactoring passes.
- The "refactor" is turning into a rewrite (most of the file's logic is now different, not just restructured) — that's a rewrite, which needs its own review/testing rigor (treat it like new code under `robust-coding-practices`), not the incremental refactor discipline.
- No tests exist and writing characterization tests is impractical (e.g. code with hard-to-control side effects) — flag this explicitly to the user as added risk rather than proceeding silently.

## Output expectations

When performing a refactor, narrate the steps as you take them (what's being extracted/renamed/moved and why) rather than presenting one large diff at the end — this lets the user follow the reasoning and stop you if a direction is wrong, and mirrors how the steps would actually be committed.
