## What it does

`test-strategy` decides which tests a change needs and writes as few of them as protect its important behaviour, at the highest seam that can observe it. It plans the tests before the code and writes them after. There is no red-green loop: the expected values are fixed from the spec before the implementation exists, so the tests can't copy its bugs, and every test is proven able to fail by a sabotage check.

## When to reach for it

Type `/test-strategy`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when a task fits: implementing a feature or ticket, deciding what tests a change needs, or writing integration or e2e tests.

| Situation | Use |
| --- | --- |
| A behaviour to build and test, with or without a spec | `test-strategy` |
| A spec or tickets, and you want the whole build run for you | [implement](https://aihero.dev/skills-implement), which drives `test-strategy` per ticket |
| A bug you can't yet reproduce | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs), which writes one reproducing test before the fix |

## Climbing down only when forced

Each **critical path** gets one test at the highest level that works. E2E covers the vital user journeys and what only the real platform shows. Integration, through a module's public interface with its real collaborators, is the default. A unit test needs a named reason: an input space too large to sample from above, a critical branch no higher seam can reach, or a higher test whose failures were hard to locate. Anything below e2e carries a one-line "why not higher".

## The sabotage check

The red step of TDD proved that a test could fail. Here that proof comes after the fact: plant the plausible mistake the test claims to catch, watch it go red at the intended assertion, then revert. A test that stays green is fixed or deleted. The same move settles whether a unit test is needed at all: break the unit, and if the higher tests already go red, it isn't written.

## Common questions

**Isn't writing tests after the code exactly what TDD warns against?**

The risk is real for agents: tests written after the code tend to agree with it, bugs included. The fix here is independence rather than order. Expected values come from the spec, written into the plan before the code exists, and never from running the code and pasting its output.

**What happens to the tests I already have?**

They stay until a new higher-level test subsumes them. The prune step then merges or deletes them: replace, don't layer.

## It's working if

- The commit or PR body lists the critical paths, the seam each is tested at, and what was deliberately left untested.
- Most new tests are integration or e2e; each unit test has its reason beside it.
- The trace shows each new test failing under sabotage before the work is called done.
- Test count grows more slowly than features do.

## Where it fits

A model-invoked discipline inside the build step of the chain (`to-spec → to-tickets → implement → code-review`). [to-spec](https://aihero.dev/skills-to-spec) agrees the seams and critical paths, [implement](https://aihero.dev/skills-implement) drives this skill per ticket, and [code-review](https://aihero.dev/skills-code-review) checks the result against a test-smell baseline. [ask-matt](https://aihero.dev/skills-ask-matt) maps the whole set.
