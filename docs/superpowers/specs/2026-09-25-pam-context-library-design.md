# Pam context library — design

**Date:** 2026-09-25
**Status:** Sections 1–2 approved in discussion. Sections 3–5 are proposals, not
yet reviewed.

## Intent

Pam currently reads one hard-set product wiki. Make her generic: the project's
context can be any mix of wiki git URLs, local folders of `.md`/`.txt`, and
artifacts the user hands over (meeting notes, notetaker transcripts, docs). Pam
keeps a registry of these sources, files new artifacts on request, and — when a
ticket arrives — reads all of them and judges what matters.

**Said by the user:** several sources at once; a sources file Pam maintains;
artifacts can be handed to her and saved; "all context is context but she should
know how to use it and what is important and what is not"; artifacts arrive both
standalone and with tickets (option C); weight is a default by type, overridable
at add time (option B); the library lives inside the project repo.

**Success:** point Pam at `C:\docs\product`, `https://github.com/org/wiki.git`,
and a pasted meeting transcript; on the next ticket she produces a cited
`context.md` drawing on all three, leading with what is authoritative and
relevant, and surfacing any disagreement between them.

**Out of scope:** non-git web URLs, binary formats (PDF, docx — Pam asks for a
text export), sharing a library across repos, automatic resolution of conflicts.

---

## 1. Layout and registry — approved

```
<repo>/.tickets/_context/
├── sources.md     registry: one entry per source
├── artifacts/     things handed to Pam, saved unchanged
├── inbox/         transient: pasted text saved by the caller, awaiting filing
├── cache/         clones of git sources
└── index.md       file tree + headings across all sources, per-source version stamp
```

`.tickets/` is already excluded via `.git/info/exclude` (Michael, intake), so the
library is private by default. No extra `.gitignore`.

`sources.md` entry:

```markdown
## product-wiki
- kind: git            # git | folder | artifact
- location: https://github.com/org/wiki.git
- type: wiki           # wiki | spec | decision | meeting-notes | transcript | chat | doc | other
- weight: authoritative   # authoritative | supporting | background
- weight-reason: default for type wiki
- dated: 2026-09-20       # artifacts only: when the content is from
- added: 2026-09-25
- last-read: 2026-09-25 @ a1b2c3d
- notes: Billing lives under commerce/, not billing/.
```

Default weights: `wiki`/`spec`/`decision` → authoritative;
`meeting-notes`/`transcript`/`doc` → supporting; `chat`/`other` → background.
A user override records its reason in `weight-reason`; Pam never changes a
user-set weight.

`notes` replaces `.tickets/_memory/pam.md`.

## 2. Add flow (`mode: add`) — approved

**Entry points:** new standalone skill `/pam-add <path | git URL | pasted text>`
at any time; Michael at intake for anything attached to a ticket. Both dispatch
Pam with `mode: add`. Pasted text is saved by the caller to `_context/inbox/`
first (agents receive paths, not content).

**Per item:**
1. Kind — git URL → `git`, cloned to `cache/<name>/`; directory → `folder`,
   read in place; single file or pasted text → `artifact`, copied byte for byte
   to `artifacts/YYYY-MM-DD-<slug>.<ext>` (inbox copy removed).
2. Dedupe — same location, or identical artifact content, updates the existing
   entry.
3. Type — from what the user said, else inferred from name and content; when
   unclear, `other` (background) and say so. Never block on type.
4. Weight — the user's override with reason, else the type default.
5. `dated` — when the content is from (meeting date in header, date in name);
   else date added, marked `(date added)`.
6. Register in `sources.md`; add to `index.md`.

**Result:** one line per item — `name · kind · type · weight · dated`. The caller
shows these to the user, who corrects by saying so or editing `sources.md`.

**Limits:** text only; never edits artifacts after filing; never writes inside
folder sources or git clones; never changes a user-set weight.

## 3. Context flow (`mode: context`) — proposed

1. Read `ticket.md`, then `sources.md`. No entries → `BLOCKED` asking what the
   project's sources are.
2. Refresh each source and record the version read:
   - `git` — `pull --ff-only`; commit sha. Never read a clone whose pull failed
     as if it were current.
   - `folder` — commit sha (+ `uncommitted changes`) if a git repo, else
     `unversioned`.
   - `artifact` — its `dated`; artifacts never change.
3. An unreadable source is listed under *Sources unavailable*. If it is
   `authoritative` → `BLOCKED` (Jim must not plan without it); otherwise carry
   on.
4. Update `index.md`: rebuild a source's section when its version moved, always
   for unversioned or dirty folders; otherwise reuse. (Replaces the per-ticket
   `wiki-index.md`, which could never be reused.)
5. Search all sources for the ticket's features, flows and terms — wide grep,
   index for neighbours, then read what matters.
6. Weigh, in order: **relevance** (would it change Jim's plan or Toby's tests?
   if not, leave it out) → **weight** (authoritative first; background only
   where nothing heavier speaks, and say so) → **recency** (among equals, newer
   is *probably* current).
7. Conflicts are reported with both citations, weights and dates, newer named —
   never settled by Pam.
8. Write `context.md`; update `last-read` on each source read.

**Return:** `STATUS: DONE` + three lines (found, conflicts, unavailable), or
`STATUS: BLOCKED` + batched questions. Unchanged vocabulary.

## 4. `context.md` format — proposed

```markdown
# Product context for <TICKET-ID>

## Sources read
| Source | Kind | Weight | Version read |
|---|---|---|---|
| product-wiki | git | authoritative | a1b2c3d |
| pricing-sync | artifact | supporting | dated 2026-09-20 |

## Sources unavailable
<source — reason; omit section if none>

## Relevant features
### <feature name>
<summary>
> Source: product-wiki · `billing/refunds.md` — "Refund window" · authoritative

## Constraints and prior decisions
<what the sources say must remain true, each cited>

## Conflicts
### <topic>
- product-wiki · `billing/refunds.md` — "Refund window" (authoritative, a1b2c3d): <says X>
- pricing-sync · lines 40–52 (supporting, dated 2026-09-20): <says Y>

Newer: pricing-sync. Not resolved — needs a decision.

## Not found
<what was searched for in every source and is genuinely absent>
```

Citations: source name + file path + heading (markdown) or line range (plain
text) + weight.

## 5. Pipeline integration — proposed

- **Michael, intake:** if the user attached notes, files or transcripts with the
  ticket, save pasted text to `_context/inbox/`, dispatch Pam `mode: add`, show
  the filed lines. Then continue.
- **Michael, context stage:** dispatch Pam `mode: context` with the ticket
  folder. The "wiki location, ask once, record in `decisions.md`" step is
  replaced: if `sources.md` is empty, ask the user for sources, run the add flow,
  then context. If `context.md` lists *Conflicts*, print them to the user as
  soon as Pam returns — they are decisions only the user can make.
- **`/pam-add` corrections:** when the user corrects a type or weight after
  filing, the skill edits that entry in `sources.md` directly and sets
  `weight-reason: user: <their words>`. Removing a source is deleting its entry.
- **Michael, final gate:** Pam's proposed notes are shown alongside other agents'
  memory proposals; accepted ones go into that source's `notes` field instead of
  `_memory/pam.md`. Pam never writes notes herself.
- **Jim:** "plan against the wiki alone" / "the wiki is silent" become "the
  context sources". A relevant entry under *Conflicts* that the plan hinges on is
  a reason to block, like a silence is.
- **`state.json`:** drop `wiki_ref` (the library path is fixed); `context` stage
  reads "Pam is gathering context".
- **README:** Pam's row, the Configuration section, and the Layout section
  (new `skills/pam-add/`).
- **Pam frontmatter:** new description covering both modes; add `Edit` to tools
  for updating `sources.md` in place.

## Risks

- **Clone size/auth.** Private git URLs need the user's existing git
  credentials; Pam does not handle auth. A failed clone surfaces as `BLOCKED`.
- **Weight drift.** Inferred types are visible in the add result and in
  `sources.md`; the user is the correction mechanism.
- **Index growth** across many sources — acceptable for markdown-scale
  libraries; revisit if a source is very large.
