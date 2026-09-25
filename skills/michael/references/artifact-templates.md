# Artifact templates

Every artifact carries its **reasoning**, not just its conclusion. The rejected
options and the "why" are the point — they are what makes the trail readable six
weeks later.

All artifacts live in `.tickets/<TICKET-ID>/`, except Pam's context library,
which is shared by every ticket and lives in `.tickets/_context/`.

---

## ticket.md — MICHAEL

```markdown
# <TICKET-ID> — <title>

**Type:** bug | feature | spike
**Source:** jira:PROJ-123 | github:<url> | pasted
**Classified because:** <one line — why this type and not another>

## Problem
<what is wrong, in the reporter's terms>

## Deliverables
- <what must be true when this is done>

## Acceptance criteria
- <testable statements; Toby maps tests to these>

## Reproduction steps (as given)
<verbatim from the ticket; empty if none were given>

## Test surface
<areas likely to need tests or to regress>

## Open questions
<anything the ticket does not answer>
```

---

## context.md — PAM

Format defined in `agents/pam.md`, under *Formats*, so Pam carries it with her.
In short: the sources read and their versions, sources unavailable, relevant
features, constraints, conflicts between sources (reported, never settled), and
what was not found. Every claim cites source name, file, and heading or line
range, plus the source's weight.

---

## sources.md — PAM

The context library's registry, at `.tickets/_context/sources.md`, shared by
every ticket. Format defined in `agents/pam.md`, under *Formats*. One entry per
source: kind (`git` | `folder` | `artifact`), location, type, weight
(`authoritative` | `supporting` | `background`) and the reason for it, dates,
and user-approved notes. The user may edit any entry.

| Type | Default weight |
|---|---|
| `wiki`, `spec`, `decision` | authoritative |
| `meeting-notes`, `transcript`, `doc` | supporting |
| `chat`, `other` | background |

A `weight-reason` starting `user:` marks a weight the user set. Pam never
changes it.

---

## repro.md — DWIGHT

```markdown
# Reproduction for <TICKET-ID>

**Outcome:** reproduced | not reproduced | supplied by user
**Attributed to:** Dwight | the user

## What was tried
1. <step> -> <observed>

## Evidence
<logs, stack traces, failing output — shortest decisive excerpt, not a dump>

## Affected code
- `path/to/file.ts:120` — <why this is implicated>

## What was ruled out
<hypotheses tested and eliminated>
```

---

## plan.md — JIM

```markdown
# Plan for <TICKET-ID>

## Approaches considered

### A. <name>  [CHOSEN]
<how it works>
**Why chosen:** <reason>

### B. <name>  [rejected]
**Why rejected:** <reason — this section is not optional>

## Change list
| File | Change | Why |
|---|---|---|

## Risks and regressions
<what this could break; Angela checks the diff against this list>

## Out of scope
<what this deliberately does not do>
```

---

## tests.md — TOBY

```markdown
# Test plan for <TICKET-ID>

**Test command:** `<resolved from package.json>`

| Test | File | Acceptance criterion | Status |
|---|---|---|---|

## Why these and not more
<what was deliberately not tested, and why>
```

---

## reviews/<agent>-<phase>-<NN>.md — written by MICHAEL

Reviewers have no write tools; they return findings and Michael persists them.

```markdown
# <ANGELA|OSCAR> — <plan|code> review, round <NN>

**Verdict:** APPROVED | CHANGES_REQUESTED | BLOCKED

## Findings
### <NN>. <one-line claim>  [blocking | non-blocking]
**Where:** `path:line` (code review only)
**Why it matters:** <consequence>
**Suggested direction:** <not a patch — reviewers do not write code>

## Considered and let go
<what was examined and judged fine — stops the next round re-raising it>

## Round <NN-1> findings
<for rounds after the first: each prior finding marked addressed | not addressed>
```

---

## questions.md — append-only

```markdown
## Q1 — PAM, stage: context, 2026-09-02T14:02Z
<question>

**Answer (user):** <verbatim>
```

Never edit an existing entry. Never answer on the user's behalf.

---

## decisions.md — append-only

```markdown
## <timestamp> — <decision>
**Decided by:** Michael | the user | Angela
**Chose:** <what>
**Because:** <why>
**Rejected:** <what lost, and why>
```

---

## worklog.md — append-only

One line per stage transition: timestamp, stage, agent, outcome.

---

## _memory/<agent>.md — proposed by agents, approved by the user

Repo-scoped, at `.tickets/_memory/`. **Durable essentials only.**

The test: *will this still be true in three months?*

- Qualifies: "Auth tests need the docker fixture running first."
- Does not: "PROJ-123 failed on line 40."

Agents never write these files. They propose lines in their result; Michael shows
the user at the final gate; only accepted lines are written. Cap each file at
roughly 40 lines so growth forces curation instead of accretion.

Pam has no `_memory` file. Her durable facts are about sources, so accepted ones
go into that source's `notes` field in `.tickets/_context/sources.md`.
