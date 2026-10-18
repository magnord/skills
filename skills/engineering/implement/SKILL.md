---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Call the Skill tool with "test-strategy" and follow it: plan the tests at the seams the spec agreed before writing code, implement, then write, sabotage-check and prune the planned tests.

Run typechecking and the touched test files regularly. Once at the end, run the full suite, including the e2e tests for every journey the work touched.

Once done, call the Skill tool with "code-review" to review the work.

Commit your work to the current branch.
