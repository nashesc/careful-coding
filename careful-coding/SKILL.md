---
name: careful-coding
description: Reduces common LLM coding mistakes - silently picking an interpretation instead of surfacing it, overengineering simple requests, drive-by refactoring unrelated code, and treating vague instructions as if they were verifiable goals. Use this for any non-trivial coding task - new features, bug fixes, refactors - especially when multiple valid approaches exist, scope is ambiguous, or you're editing existing code. Skip it for trivial one-liners and obvious typo fixes.
---

# Careful Coding

Four checks to run before and during any non-trivial coding task. Source: Andrej Karpathy's observations that LLMs "make wrong assumptions and just run along with them," "overcomplicate code and APIs," and "change/remove comments and code they don't sufficiently understand as side effects."

**Tradeoff:** these checks bias toward caution over speed. For trivial tasks (one-liners, obvious fixes), don't apply all four ceremonially - use judgment.

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

- State assumptions explicitly before implementing. If genuinely uncertain on something consequential, ask instead of guessing.
- If multiple reasonable interpretations exist, present them - don't silently pick one.
- If a simpler approach exists than what was asked for, say so.
- If something is unclear, name what's confusing rather than coding around it.

Example - request: "Add a feature to export user data."
A wrong-by-default response writes a 30-line function that silently decides to export *all* users, picks the file location, and guesses which fields are safe to include. The better response flags that scope (all users vs. filtered - privacy implications), format (download vs. background job vs. API), and fields (some may be sensitive) are all open questions, proposes the simplest unblocked option (a paginated API endpoint), and asks before building the rest.

## 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code (no strategy pattern for one discount calculation).
- No "flexibility" or config options that weren't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

Test: would a senior engineer call this overcomplicated? If yes, simplify.

Example - request: "Add a function to calculate discount." The wrong response builds an `ABC` strategy class, a config dataclass, and a calculator object for a single percentage calculation. The right response is one function: `calculate_discount(amount, percent)`. Add the abstraction later, when a second discount type actually shows up.

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

- Don't "improve" adjacent code, comments, or formatting while fixing something else.
- Don't refactor things that aren't broken.
- Match existing style even if you'd personally do it differently (quote style, type hints or lack thereof, naming conventions already in the file).
- If you notice unrelated dead code, mention it - don't delete it unasked.
- Do remove imports/variables/functions that *your own* change made unused. Don't remove pre-existing dead code.

Test: every changed line should trace directly to the user's request. "Fix the bug where empty emails crash the validator" means touching the email-emptiness check - not also rewriting the email regex, adding username-length validation, and reformatting whitespace nobody complained about.

## 4. Goal-Driven Execution

Define success criteria. Loop until verified, not until it "looks done."

Transform vague instructions into verifiable goals before starting:

| Instead of... | Transform to... |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

For multi-step work, state a brief plan with a verify step attached to each item:
```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
```

Example - "The sorting breaks with duplicate scores." Don't jump straight to changing the sort key. First write a test that reproduces the non-deterministic ordering, confirm it fails, then add the stable secondary sort key and confirm the test now passes consistently. Reproducing first is what proves you fixed the reported bug rather than a different one you assumed was there.

## Further detail

`references/examples.md` has the full before/after code walkthroughs (5 examples, ~430 lines) this file's condensed versions are drawn from, plus the anti-pattern summary table. Read it when you need to *show* someone a fuller wrong-vs-right diff - e.g. explaining a violation in detail, or writing a longer review comment - not on every trigger.

## Signals this is working

- Diffs contain only changes that trace to the request.
- Fewer rewrites caused by initial overcomplication.
- Clarifying questions surface before implementation, not after a wrong guess.
