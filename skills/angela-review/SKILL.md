---
name: angela-review
description: Get an adversarial architecture and regression review of a design, plan, or diff, outside the Dunder Mifflin pipeline. Use when the user asks whether an approach is sound, wants a design challenged, asks "is this even needed", or wants a finished change checked for regressions against its intent. For line-by-line code quality use oscar-review instead.
---

# Angela, standalone

Runs Angela against an arbitrary target instead of a ticket folder. Same lens as
the pipeline: architecture, necessity, and regressions — not code craft.

## Steps

1. **Establish the target.** If the user did not say, ask — do not guess:
   - a design or plan (a document, or described in chat)
   - the current diff (`git diff`)
   - a specific commit range or branch
   - a set of files

2. **Establish the intent.** Angela judges a change against what it was supposed
   to do. If there is no stated intent, ask the user for it in one line. Reviewing
   against an intent you invented is worthless.

3. **Dispatch the `angela` agent** with:
   - `mode: plan` for a design or plan, `mode: code` for a diff
   - the target and the intent
   - a note that this is a standalone run, so there is no ticket folder and she
     should return her findings in full rather than expecting Michael to persist
     them

4. **Print her verdict and every finding.** Do not summarise away her reasoning —
   the "considered and let go" section is part of the value.

5. **Offer to save** the review to a file if the user wants it kept. Do not write
   one unasked.

## Constraints

Angela has no write tools and does not fix what she finds. If the user wants the
findings acted on, that is a separate step they ask for explicitly.
