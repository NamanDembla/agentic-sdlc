---
name: oscar-review
description: Get a line-by-line senior-engineer code critique of a diff, branch, or set of files, outside the Dunder Mifflin pipeline. Use when the user asks for a code review, wants their changes critiqued before pushing, or asks what is wrong with a piece of code. For architecture and "was this the right approach" use angela-review instead.
---

# Oscar, standalone

Runs Oscar against an arbitrary target instead of a ticket folder. Same lens as
the pipeline: correctness, silent failure, type honesty, reuse, simplification,
test quality, and fit — not architecture.

## Steps

1. **Establish the target.** Default to the unstaged working tree (`git diff`).
   If the user meant something else — a branch, a commit range, specific files —
   use that. If it is ambiguous and the diff is large, ask.

2. **Establish the intent** in one line if the user has not said what the change
   is meant to do. Oscar reviews execution, and execution is judged against a
   goal.

3. **Dispatch the `oscar` agent** with the target and intent, and a note that
   this is a standalone run with no ticket folder, so he should return his full
   review rather than expecting Michael to persist it.

4. **Print his verdict and every finding**, each with its `path:line` and its
   blocking / non-blocking mark.

5. **Offer to fix.** Unlike the pipeline — where only Kevin writes code — a
   standalone review may be followed by fixes if the user asks. Ask first; do not
   start editing off the back of a review they only wanted to read.

## Constraints

Oscar has no write tools and never writes the patch himself. Any fixing happens
after, and only on request.
