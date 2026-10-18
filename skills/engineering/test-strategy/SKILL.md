---
name: test-strategy
description: Choose and write the fewest tests that protect a change's important behaviour, at the highest seam that can observe it. Use when implementing a feature, ticket or fix, deciding which tests a change needs, or when the user asks for integration or e2e tests.
---

# Test strategy

Plan the tests before the code, write them after, and keep only as many as protect the behaviour that matters. There is no red → green loop. Its two real benefits are kept by other means: **expected values fixed from the spec before the code exists**, so tests can't inherit the code's bugs, and the **sabotage check**, so every test is proven able to fail.

Read `GLOSSARY.md` (if it exists) so test names use the project's domain language, and respect ADRs in the area you're touching. Read the repo's agent guide for its test commands. When no harness exists at the seam you need, say so and propose building it; don't quietly drop to a lower level.

## Levels

- **E2E**: drives the product through its outermost interface (UI, CLI, HTTP) with the real wiring. Only true externals are replaced, by local stand-ins.
- **Integration**: exercises a module or subsystem through its public interface with its real in-process collaborators, and local stand-ins for I/O (in-memory database, temp directory, fake clock, recorded fixtures).
- **Unit**: one function or a small cluster, in isolation.

## 1. Test plan, before code

List the **critical paths**: the behaviours the acceptance criteria name, plus code where a plausible mistake would hurt users or corrupt data. Weigh risk as criticality, churn and complexity together. Everything else goes untested by default, and the plan says so.

For each critical path record:

- its **seam** (take it from the spec's testing decisions when they name one) and level;
- its **expected outcome**, as concrete values taken from the spec, a worked example or an oracle. Writing these down before the implementation exists is what stops the tests from copying its bugs;
- for anything below e2e, one line saying **why not higher**.

Climb down a level only when forced:

1. **E2E** for user journeys and for what only the real platform shows (configuration, permissions, native bridges, device I/O), when a harness exists and the test can be made deterministic. Keep these to the vital journeys; they are the slowest and flakiest tests you own.
2. **Integration** is the default for everything else, and for what e2e cannot observe (an internal branch, a timeout, a storage failure) or would need many slow runs to cover.
3. **Unit** only when one of these holds, and the plan names which:
   - the logic is pure or nearly so, and its input space has many cases or boundaries that a higher test would only sample (parsers, arithmetic, alignment, scheduling, state machines);
   - a critical branch, often error handling, can't be reached reliably through a higher seam;
   - a higher test failed and the cause was hard to locate, so the unit needs fast, local feedback.

   Write the cases as one table-driven or property-based test, not a test per example.

## 2. Implement

Build against the spec, not against tests, in small steady steps. Keep the feedback loop fast: typecheck, run the existing tests that touch the area, and exercise the behaviour directly (run the app, a script, a request).

## 3. Write the planned tests

- Add an assertion or a case to an existing test before writing a new one.
- Assert on observable outcomes through the seam, using the expected values from the plan. See [tests.md](tests.md) for good and bad tests and [mocking.md](mocking.md) for where stand-ins belong.
- Never take an expected value from running the code and pasting its output; that is the test-after tautology. When the plan's value turns out wrong, re-derive it from the spec and say so. For spec-driven or invariant-heavy code (parsers, codecs, protocols, numeric pipelines), pick oracles from [oracles.md](oracles.md).
- Control time, randomness, network and data, so a red test always means the behaviour broke. A flaky test is quarantined until fixed, never retried to green.

## 4. Sabotage check

Every new test earns its place by going **red under sabotage**: introduce the plausible mistake it claims to catch (drop the branch, flip the comparison, swap operands, revert the fix), confirm with `git diff` that the change applied, run the test, and check it fails at the assertion you meant. Then revert and see it pass. A test that stays green is fixed or deleted.

When the repo has a mutation tool configured, run it on the changed lines instead. Treat survivors on critical paths as missing tests, and ignore low-value mutants (logging, defaults, equivalent rewrites) rather than writing change-detector tests to kill them.

The same check settles unit tests: sabotage the unit and run the higher tests. If they go red, the unit test isn't written. Coverage from higher tests only counts when a sabotage proves it is checked, not just executed.

## 5. Prune

A test stays only if it protects a critical path, or catches a sabotage, that no other test does. Merge or delete the rest, including older tests that the new higher-level tests now subsume: replace, don't layer. A lower-level duplicate survives only when it makes failures much faster to locate.

## Rules for the tests themselves

- Never edit, skip, weaken or delete an existing test without saying so and why.
- Never special-case the code to satisfy a test.
- "Tests pass" is claimed only alongside the command and its output.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private functions, or verifies through a side channel. The tell: it breaks on a refactor that kept behaviour.
- **Tautological**: the expected value is recomputed the way the code computes it, or copied from the code's output, so the test passes by construction.
- **Unit by habit**: a unit test with no named reason.
- **Coverage padding**: tests that raise a percentage on code no critical path runs through.
- **Retry to green**: a flaky test kept alive by retries instead of being made deterministic.

## Done when

Every critical path has a test at its seam that went red under sabotage, every test below e2e names its reason, anything deliberately left untested is listed, and the full suite passes. Put the plan, the sabotage results and the untested list in the commit or PR body.
