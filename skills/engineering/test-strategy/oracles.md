# Oracles and discriminating inputs

For code built from a spec or bound by stated invariants.

## Oracles

Where the expected value comes from, strongest first:

- **Spec literal**: a worked example, test vector or table in the spec.
- **Reference implementation**: an existing library or tool that produces the answer. Compare outputs on the same inputs (differential testing).
- **Fresh-context derivation**: a subagent given only the spec and the input computes the expected output by hand. It shares no helper code with the implementation, and it checks that each input is legal under the spec. Use it when you wrote the implementation and the spec has no literal for the case.
- **Round trip**: `decode(encode(x)) == x`. It only catches bugs that aren't symmetric, so pair it with another oracle for one direction.
- **Metamorphic relation**: a transformation of the input with a known effect on the output (transpose by k semitones and relative findings stay identical; reorder independent records and the totals don't change). Choose it so the risky code sits between input and output.

## Discriminating inputs

For every input, ask "which plausible mistake does this catch?" If the answer is none, change the input.

- **Asymmetric inputs**: distinct values wherever two could be confused, and non-palindromic sequences for anything that reorders.
- **Both sides of every boundary**: the last value that is in and the first that is out.
- **One reason to fail**: each case fails only for the mistake it targets. A typo, reserved word or shadowed name in the input is a bug in the test.

## Generated inputs

Where the input space is large, use the ecosystem's property-based library (fast-check, Hypothesis, proptest, jqwik). Steer generators toward structurally valid, interesting inputs that reach deep paths, and assert a property or an oracle result, not just "did not throw". One property test usually replaces a page of examples.
