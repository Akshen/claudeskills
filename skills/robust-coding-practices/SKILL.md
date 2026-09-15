---
name: robust-coding-practices
description: Enforces professional software engineering standards on every coding task — clean architecture, comprehensive test coverage, security hardening, error handling, and performance. Use this skill whenever writing, editing, reviewing, or debugging code in any language, whether it's a new feature, a bug fix, a script, or a full application. Applies even when the user doesn't explicitly ask for "best practices," "tests," or "security" — these are the default bar for all code output, not optional extras.
---

# Robust Coding Practices

A cross-language standard for producing code that is correct, secure, tested, and maintainable by default. Apply this on every coding task, not just when explicitly asked for "quality" or "production-ready" code.

## Core Principle

Treat every piece of code as if it will run in production, be read by someone else, and be attacked by someone hostile. Optimize for correctness and clarity first, cleverness never.

---

## 1. Before Writing Code

- Restate the requirement in your own words and identify edge cases (empty input, null/None, zero, negative numbers, huge input, unicode, concurrent access, network failure, malformed data).
- Check for existing conventions in the repo (linter config, naming style, test framework, folder structure) and match them rather than imposing your own.
- Prefer the smallest change that correctly solves the problem — avoid speculative abstraction ("YAGNI").
- If a dependency is needed, prefer well-maintained, widely-used libraries over hand-rolled solutions for solved problems (crypto, parsing, date handling).

## 2. Code Quality Checklist

- **Naming**: descriptive, unambiguous names; no single-letter variables outside tight loops/indices.
- **Functions**: single responsibility, short enough to read in one screen; extract helpers rather than nesting deeply.
- **DRY, but not premature**: deduplicate real repetition; don't abstract after only one occurrence.
- **Immutability by default**: prefer `const`/`final`/immutable data structures unless mutation is required.
- **No magic numbers/strings**: name constants.
- **Comments explain *why*, not *what***; code should be self-explanative for the "what."
- **Consistent formatting**: run the project's formatter/linter (e.g. `prettier`, `black`, `gofmt`, `rustfmt`) before considering work done.
- **Type safety**: use type hints/annotations, static typing, or strict mode where the language supports it (TypeScript strict, Python type hints + mypy, etc.).

## 3. Error Handling & Robustness

- Never silently swallow exceptions — log, handle, or propagate with context.
- Fail fast on programmer errors (assertions, invariant checks); handle gracefully on expected failure modes (bad input, network timeout).
- Validate all external input (function args from callers you don't control, API payloads, file contents, env vars) at the boundary.
- Use specific exception/error types, not bare `except:`/`catch (Exception e)`.
- Always include cleanup for resources (files, sockets, DB connections, locks) — use `try/finally`, context managers (`with`), `defer`, or RAII equivalents.
- Design for idempotency where operations may be retried (payments, writes, message processing).
- Add timeouts and retry-with-backoff for any network or I/O call; never block indefinitely.
- Handle concurrency explicitly: avoid shared mutable state, guard critical sections, watch for race conditions and deadlocks.

## 4. Security Checklist

Apply regardless of whether the user asked for a "security review." Never write code that:

- Concatenates untrusted input into SQL, shell commands, or file paths → use parameterized queries, prepared statements, `subprocess` argument arrays (never `shell=True` with untrusted input), and path sanitization/allow-lists.
- Renders untrusted input into HTML/JS without escaping → use templating engines' auto-escaping; never `innerHTML` or `dangerouslySetInnerHTML` with unsanitized data.
- Hardcodes secrets, API keys, passwords, or tokens → load from environment variables or a secrets manager; add `.env` and credential files to `.gitignore`.
- Uses weak or homemade cryptography → use vetted libraries (`bcrypt`/`argon2` for passwords, standard TLS, established JWT libraries); never roll your own crypto or use MD5/SHA1 for security purposes.
- Deserializes untrusted data with unsafe methods (`pickle.loads`, `eval`, `yaml.load` without `SafeLoader`) → use safe/structured serialization (JSON, `yaml.safe_load`).
- Skips authentication/authorization checks on any new endpoint or resource access, including indirect object references (verify the caller owns/can access the specific resource ID, not just that they're logged in).
- Logs sensitive data (passwords, tokens, PII, full card numbers) in plaintext.
- Trusts client-side validation alone — always validate/authorize again server-side.
- Leaves default-open CORS, debug modes, or verbose stack traces enabled in production-facing code.
- Introduces dependencies without a quick sanity check on maintenance status / known CVEs for anything security-sensitive.

For each new piece of code that touches user input, auth, money, or data storage, explicitly ask: *"What happens if this input is malicious, and what's the worst this code path could be tricked into doing?"*

## 5. Testing

Every non-trivial function or module should ship with tests, written in the project's existing test framework (or a sensible default: `pytest`, `jest`/`vitest`, `go test`, `JUnit`, etc.).

**Coverage to include:**
- Happy path — typical, expected input.
- Boundary conditions — empty/null, zero, min/max, single-element collections, off-by-one edges.
- Invalid/malformed input — wrong types, out-of-range values, malicious strings (for security-sensitive code).
- Error paths — assert the correct exception/error is raised, not just that "it doesn't crash."
- Concurrency/idempotency cases where relevant (double-submit, race conditions).
- Regression tests for any bug fixed — a failing test that reproduces the bug before the fix, passing after.

**Test hygiene:**
- Tests should be deterministic (no reliance on real time, real network, or random seeds without fixing the seed).
- Mock/stub external dependencies (DB, network, filesystem, clock) so unit tests are fast and isolated; keep a smaller set of integration tests for real wiring.
- One logical assertion focus per test; descriptive test names that state the scenario and expected outcome (`test_withdraw_raises_when_balance_insufficient`, not `test_1`).
- Keep tests independent — no test should depend on another test's side effects or execution order.
- After writing code, actually run the test suite (and the linter/type-checker) rather than asserting it should pass — use the bash/execution tool available in this environment.

## 6. Performance & Scalability (proportionate to the task)

- Know the algorithmic complexity (Big-O) of the approach; avoid accidental O(n²)+ on data that can grow.
- Avoid N+1 queries — batch or join instead of looping DB calls.
- Stream/paginate large datasets rather than loading everything into memory.
- Don't optimize prematurely — write clear code first, profile before micro-optimizing, and note explicitly when you're making a deliberate performance trade-off.

## 7. Documentation

- Public functions/classes/modules get a docstring/doc-comment: purpose, parameters, return value, exceptions raised.
- Non-obvious "why" decisions get an inline comment (a workaround for a library bug, a regulatory constraint, a perf trade-off).
- If the change is user-facing or alters a public API/CLI, update the README/usage docs in the same pass.

## 8. Self-Review Pass (do this before presenting code as done)

Walk through this list against the code you just wrote:

1. Does it handle empty/null/zero/negative/huge inputs?
2. Can any untrusted input reach a command, query, filesystem path, or template unescaped?
3. Are there hardcoded secrets or credentials anywhere?
4. Are all resources (files, connections, locks) guaranteed to be released?
5. Are errors caught at the right level with useful context, not swallowed?
6. Did I write tests covering happy path, edges, and failure modes — and run them?
7. Did I run the linter/formatter/type-checker?
8. Would another engineer understand this in six months without asking me?
9. Is there dead code, debug prints, or commented-out blocks left behind? Remove them.
10. If this fails in production at 3am, does the error message/log give the on-call engineer enough to act on?

If the answer to any of these is "no" or "unsure," fix it or flag it explicitly to the user rather than presenting the code as complete.

## 9. When Trade-offs Are Necessary

Real constraints (deadline, prototype/throwaway script, explicit user request for something quick) can justify skipping parts of this checklist. When that happens, say so explicitly — e.g. "Skipping input validation here since this is a one-off local script, not exposed to untrusted input" — rather than silently cutting corners. Never silently skip security-critical items (secrets handling, injection safety, auth checks) even for "quick" code that touches real user data or is reachable over a network.
