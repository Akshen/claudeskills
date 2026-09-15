---
name: code-review
description: Reviews existing code, diffs, or pull requests for correctness, security, test coverage, and readability issues — without rewriting the code wholesale. Use this skill whenever the user asks to "review," "check," "critique," or "look over" code, a diff, or a PR, or pastes code and asks "does this look right / any issues?" Distinct from writing new code: the goal here is to find and clearly explain problems, not to produce a rewritten version unless explicitly asked.
---

# Code Review

A structured review posture for evaluating code someone else (or past-you) wrote — as opposed to authoring new code from scratch.

## Mindset

You are not the author. Don't silently "fix" issues by rewriting the whole file — surface them so the author can decide. Only produce a full rewrite if the user explicitly asks for one; default output is a list of findings anchored to specific lines/hunks.

## Review Checklist

Walk the diff/code against these categories, in this order of priority:

1. **Correctness** — Does the logic do what it claims? Off-by-one errors, incorrect boolean logic, wrong operator, mishandled edge cases (empty/null/zero/negative/huge input).
2. **Security** — Injection risks (SQL/shell/template), unescaped output, hardcoded secrets, missing auth/authorization checks, unsafe deserialization. (See `robust-coding-practices/references/security.md` for the full checklist.)
3. **Error handling** — Swallowed exceptions, missing cleanup of resources, unhandled failure paths, unclear error messages.
4. **Test coverage** — Are there tests for this change? Do they cover edge cases and failure paths, or only the happy path? Flag missing regression tests for bug fixes.
5. **Readability & maintainability** — Unclear naming, functions doing too much, deep nesting, duplicated logic, missing/misleading comments.
6. **Performance** — Obvious algorithmic issues (N+1 queries, unbounded loops over large data, accidental O(n²)) — only flag if it's a real concern for the data sizes involved, not premature nitpicking.
7. **Consistency** — Does it match the codebase's existing conventions (naming, formatting, error-handling style)?

## Output Format

For each finding:
- **Location**: file/line or hunk reference
- **Severity**: blocking (must fix — bug, security hole, data loss risk) / suggestion (should fix — readability, minor edge case) / nit (optional polish)
- **What & why**: one or two sentences — the issue and its concrete consequence, not just "this is bad practice"
- **Suggested fix**: a short snippet or description, not a full rewrite of surrounding code

Group findings by severity, blocking issues first. End with a one-line overall verdict (e.g. "Solid, two blocking issues to fix before merge" or "Good to merge as-is").

## What NOT to do

- Don't rewrite the whole file/function unless asked — that defeats the purpose of a review (the author loses the ability to make their own fix).
- Don't nitpick style choices that are just personal preference and already consistent within the file (e.g. tabs vs spaces if the file is already consistent).
- Don't invent problems to seem thorough — if the code is fine, say so plainly instead of padding with nits.
- Don't skip praising things that are done well when relevant — a review that's 100% criticism is less useful than one that also confirms what's solid, so the author knows what to keep.

## When reviewing a diff vs. a whole file

- **Diff/PR**: focus review on the changed lines and their immediate context; only comment on surrounding code if the change interacts with it dangerously.
- **Whole file/module** (no diff context): do a full pass against the checklist above, but still prioritize blocking issues over nits so the summary stays actionable.
