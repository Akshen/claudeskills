# Testing Reference

Deeper testing guidance beyond the basic checklist in `robust-coding-practices/SKILL.md` §5. Read this when the code under test parses untrusted input, has complex branching logic, or when you want to sanity-check whether existing tests are actually strong.

## Example-based vs. property-based testing

Example-based tests (the default — "given input X, expect output Y") are right for most code. Reach for **property-based testing** when:
- The function parses, serializes, or transforms data (parsers, encoders, validators) — properties like "parse(serialize(x)) == x" catch cases you wouldn't think to hand-write.
- There's a clear invariant that should hold for *all* valid inputs, not just a few examples (e.g. "sorting never changes the list's length," "the output is always within some bound").
- Input space is large enough that hand-picked examples likely miss edge cases (numeric ranges, string parsing, concurrent orderings).

Libraries: `hypothesis` (Python), `fast-check` (TypeScript/JS), `QuickCheck`/`proptest` (Rust), `gopter` (Go).

Property-based tests don't replace example-based tests — keep a few concrete examples for readability/documentation, and add property tests for the invariants.

## Fuzz testing

For anything parsing untrusted or externally-sourced input (file formats, network protocols, user-uploaded data), consider fuzz testing — feeding random/malformed input and checking for crashes, hangs, or memory issues rather than just wrong output. Worth the setup cost specifically for security-sensitive parsers, not general application code.

## Is the test suite actually strong? (mutation-testing mindset)

A test suite with high line coverage can still be weak if it never asserts on the *values* that matter. Sanity-check by asking:
- If I introduced a one-character bug here (flipped a `<` to `<=`, changed `+` to `-`), would any test actually fail? If not, the test isn't really testing that logic — it's just executing it.
- Tools that automate this check: `mutmut`/`cosmic-ray` (Python), `Stryker` (JS/TS), `cargo-mutants` (Rust). Worth running occasionally on critical modules, not every PR.
- A quick manual version: read the test's assertions and ask whether they'd catch the specific bug you'd be most worried about in this code, not just "does it run without throwing."

## Test doubles — mocking, stubbing, faking

- **Mock** what's slow, non-deterministic, or external (network calls, real DB, system clock, random number generation) so unit tests are fast and repeatable.
- Don't over-mock — if you mock so much that the test only checks "did I call the mock correctly," it stops testing real behavior. Prefer a small number of integration tests with real wiring (real DB in a test container, real HTTP call to a local test server) to catch what over-mocked unit tests miss.
- Fix random seeds and freeze/mock the clock rather than special-casing "flaky because of timing" tests.

## Test organization

- Mirror the source structure (`src/foo/bar.py` → `tests/foo/test_bar.py`) so tests are easy to locate.
- Use the framework's grouping/describe-blocks to organize by scenario, not just by function name — makes failures easier to scan.
- Keep fixtures/setup minimal and explicit in the test itself where possible; heavily shared setup across many tests can hide what's actually being tested and makes failures harder to diagnose.

## Regression tests specifically

Every bug fix should come with a test that:
1. Fails against the old (buggy) code — actually verify this if you're not 100% sure, don't just assume
2. Passes against the fix
3. Is named/commented to reference what bug it guards against (a ticket number, or a one-line description), so future readers know why this specific case is being tested

## Coverage as a signal, not a target

Line/branch coverage percentage is a floor-level signal ("this code ran at least once during tests"), not proof of correctness. Don't chase a coverage number by writing tests with no meaningful assertions just to touch a line — that's worse than the line being untested, because it creates false confidence.
