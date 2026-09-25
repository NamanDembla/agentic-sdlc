# Pam Context Library Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace Pam's single hard-set wiki with a per-repo context library of git sources, local folders and filed artifacts, which she weighs by relevance, weight and recency.

**Architecture:** Pam gains two modes — `add` (file sources into `.tickets/_context/`) and `context` (read every registered source, write a cited and weighed `context.md`). A new standalone `pam-add` skill and Michael's intake both drive `add`; Michael's context stage drives `context`. Everything else in the pipeline only changes its wording from "wiki" to "context sources".

**Tech Stack:** Claude Code plugin — markdown agent and skill definitions. No runtime code. Verification is `grep` for required content plus a smoke run of the real agent via `claude --plugin-dir`.

**Spec:** `docs/superpowers/specs/2026-09-25-pam-context-library-design.md`

## Global Constraints

- Repo: `C:\Personal Data\agentic-sdlc`. Work on branch `pam-context-library`, not `main`.
- Library root is exactly `.tickets/_context/` with `sources.md`, `artifacts/`, `inbox/`, `cache/`, `index.md`.
- Kinds: `git | folder | artifact`. Types: `wiki | spec | decision | meeting-notes | transcript | chat | doc | other`. Weights: `authoritative | supporting | background`.
- Default weights: `wiki`/`spec`/`decision` → authoritative; `meeting-notes`/`transcript`/`doc` → supporting; `chat`/`other` → background.
- Pam's return vocabulary stays `STATUS: DONE` / `STATUS: BLOCKED`.
- Pam never writes outside `.tickets/`, never edits a filed artifact, a folder source or a git clone, never changes a user-set weight, never writes `notes` herself.
- Artifacts are named `YYYY-MM-DD-<slug>.<ext>`.
- Match the existing voice: second person to the agent, short imperative sentences, bold for the rule that matters.

## Review Focus

1. **The same artifact filed twice** — expect one entry, not two. Pinned in Task 6 step 4.
2. **Re-adding a source whose weight the user set, without a weight this time** — expect the user's weight and reason to survive. Pinned in Task 6 step 5.
3. **An authoritative git source whose pull fails** — expect `BLOCKED`, not a context built from the stale clone. Pinned in Task 6 step 8.
4. **A context run with no registered sources** — expect `BLOCKED` asking for sources, not a hunt for a wiki. Pinned in Task 6 step 7.
5. **Pasted text with no date in it** — expect `dated: <today> (date added)`. Pinned in Task 6 step 3.

---

### Task 1: Templates and state schema

**Files:**
- Modify: `skills/michael/references/artifact-templates.md` (line 7; the `context.md — PAM` section, lines ~41–63; the `_memory` section, lines ~188–199)
- Modify: `skills/michael/references/state-schema.md` (lines 22, 37, 44)

**Interfaces:**
- Produces: the `sources.md` entry format and the `context.md` format that Tasks 2–4 reference by name ("the template in `skills/michael/references/artifact-templates.md`").

- [ ] **Step 1: Branch**

```bash
cd "C:/Personal Data/agentic-sdlc" && git checkout -b pam-context-library
```

- [ ] **Step 2: Run the check to see it fail**

```bash
grep -c "## sources.md — PAM" skills/michael/references/artifact-templates.md; grep -c "wiki_ref" skills/michael/references/state-schema.md
```
Expected: `0` then `2`.

- [ ] **Step 3: Replace line 7 of `artifact-templates.md`**

Old: ``All artifacts live in `.tickets/<TICKET-ID>/`.``
New:
```markdown
All artifacts live in `.tickets/<TICKET-ID>/`, except Pam's context library,
which is shared by every ticket and lives in `.tickets/_context/`.
```

- [ ] **Step 4: Replace the whole `## context.md — PAM` section** (from its heading up to, not including, the `---` before `## repro.md — DWIGHT`) with:

````markdown
## context.md — PAM

```markdown
# Product context for <TICKET-ID>

## Sources read
| Source | Kind | Weight | Version read |
|---|---|---|---|
| <name> | git \| folder \| artifact | authoritative \| supporting \| background | <sha> \| <sha>, uncommitted changes \| unversioned \| dated <date> |

## Sources unavailable
<source — reason. Omit the section when there are none.>

## Relevant features
### <feature name>
<summary>
> Source: <name> · `<path>` — "<heading>" · <weight>

## Constraints and prior decisions
<what the sources say must remain true, each cited>

## Conflicts
### <topic>
- <name> · `<path>` — "<heading>" (<weight>, <version>): <what it says>
- <name> · `<path>` lines <a>–<b> (<weight>, <version>): <what it says>

Newer: <name>. Not resolved — needs a decision.

## Not found
<what was searched for in every source and is genuinely absent — this matters>
```

Every claim carries a citation: source name, file path, and heading — or a line
range for plain text — plus the source's weight. An uncited claim is a guess, and
guesses are forbidden.

---

## sources.md — PAM

The context library's registry, at `.tickets/_context/sources.md`. One entry per
source. Pam writes entries; the user may edit any of them.

```markdown
# Context sources

## <name>
- kind: git | folder | artifact
- location: <git URL | absolute folder path | artifacts/<YYYY-MM-DD>-<slug>.<ext>>
- type: wiki | spec | decision | meeting-notes | transcript | chat | doc | other
- weight: authoritative | supporting | background
- weight-reason: default for type <type> | user: <their words>
- dated: <YYYY-MM-DD> | <YYYY-MM-DD> (date added)     # artifacts only
- added: <YYYY-MM-DD>
- last-read: <YYYY-MM-DD> @ <version> | never
- notes: <durable facts about this source, user-approved; empty if none>
```

| Type | Default weight |
|---|---|
| `wiki`, `spec`, `decision` | authoritative |
| `meeting-notes`, `transcript`, `doc` | supporting |
| `chat`, `other` | background |

A `weight-reason` starting `user:` marks a weight the user set. Pam never
changes it.
````

- [ ] **Step 5: In the `_memory/<agent>.md` section, append after its last paragraph:**

```markdown

Pam has no `_memory` file. Her durable facts are about sources, so accepted ones
go into that source's `notes` field in `.tickets/_context/sources.md`.
```

- [ ] **Step 6: Edit `state-schema.md`**

Delete the line `  "wiki_ref": "~/.cache/dunder-mifflin/product-wiki",`.
Delete the table row ``| `wiki_ref` | local path to the pulled wiki clone |``.
Replace `context       Pam is reading the wiki` with `context       Pam is gathering context from the library`.

- [ ] **Step 7: Run the check to see it pass**

```bash
grep -c "## sources.md — PAM" skills/michael/references/artifact-templates.md; grep -c "wiki_ref" skills/michael/references/state-schema.md; grep -c "Wiki version" skills/michael/references/artifact-templates.md
```
Expected: `1`, `0`, `0`. The JSON example in `state-schema.md` stays valid: the line now before `"updated"` is the `gates` object's closing `},`.

- [ ] **Step 8: Commit**

```bash
git add skills/michael/references && git commit -m "Add sources.md template and multi-source context.md format"
```

---

### Task 2: Rewrite Pam

**Files:**
- Modify (full replace): `agents/pam.md`

**Interfaces:**
- Consumes: `sources.md` and `context.md` templates from Task 1.
- Produces: `pam` agent accepting `mode: add` (items: paths/URLs + user remarks) and `mode: context` (ticket folder). Returns `STATUS: DONE` with one line per item `name · kind · type · weight · dated` (add) or three summary lines (context); or `STATUS: BLOCKED` with questions.

- [ ] **Step 1: Run the check to see it fail**

```bash
grep -c "## Mode: add\|## Mode: context\|_context/" agents/pam.md
```
Expected: `0`.

- [ ] **Step 2: Replace `agents/pam.md` with exactly:**

````markdown
---
name: pam
description: Keep the project's context library and gather product context for a ticket from it. Two modes - mode add, to file new sources (local folders, git wiki URLs) and artifacts (notes, transcripts, docs) into .tickets/_context/; and mode context, to read every registered source and write a cited, weighed context.md for a ticket. Dispatched by Michael at intake (add) and the context stage (context), and by the pam-add skill. Not for reading source code - Pam reads the context library only.
tools: Read, Grep, Glob, Bash, Write, Edit
model: sonnet
color: yellow
---

You are Pam. You sit at the desk everyone walks past and you know where
everything is. You keep the filing cabinet — every wiki, note, transcript and doc
this project has been handed — and when a ticket comes in, you find what it
already says, so nobody downstream has to guess or go looking.

All context is context, but it is not all equal. A spec outranks a chat excerpt;
last week's decision outranks last year's page on the same subject. Knowing the
difference is most of your job.

You are dispatched in one of two modes. Your prompt says which.

## The library

Everything lives in `.tickets/_context/`:

```
sources.md     the registry — one entry per source
artifacts/     things you were handed, saved as received
inbox/         pasted text the caller saved for you to file
cache/         clones of git sources
index.md       file tree and headings of every source, with the version each
               section was built from
```

The `sources.md` format is in `skills/michael/references/artifact-templates.md`.
Create any part of the library that is missing.

| Kind | What it is | How you read it |
|---|---|---|
| `git` | a wiki or docs repo URL | cloned into `cache/<name>/`, pulled fresh every context run |
| `folder` | a local directory the user keeps up to date | read in place, never copied |
| `artifact` | a single file or pasted text handed to you once | copied into `artifacts/`, never edited afterwards |

Every entry has a `type` and a `weight`. Default weight by type:

| Type | Default weight |
|---|---|
| `wiki`, `spec`, `decision` | `authoritative` |
| `meeting-notes`, `transcript`, `doc` | `supporting` |
| `chat`, `other` | `background` |

A `weight-reason` starting `user:` marks a weight the user set. **Never change
it** — not even when re-filing the same source without a weight. If you think
it is wrong, say so in your result.

You read text: markdown, plain text, and similar. If handed something you cannot
read as text, do not file it — ask for a text export.

## Mode: add

Your prompt names one or more items — local paths or git URLs — plus anything
the user said about them, verbatim.

For each item:

1. **Decide the kind.** A git URL (`https://…`, `git@…`, ending `.git`) is `git`
   — clone it into `cache/<name>/`. A directory is `folder`. A single file is an
   `artifact` — copy it to `artifacts/<YYYY-MM-DD>-<slug>.<ext>` byte for byte,
   and delete the `inbox/` copy if it came from there.
2. **Check for duplicates.** If the location is already registered, or an
   artifact with identical content is already filed, update that entry instead
   of adding another.
3. **Decide the type.** Use what the user said. Otherwise infer it from the name
   and content. When it is genuinely unclear, choose `other` and say so — never
   guess a heavier type, and never block over type.
4. **Set the weight** — the user's, if they gave one, with
   `weight-reason: user: <their words>`; otherwise the default, with
   `weight-reason: default for type <type>`.
5. **Date it.** `dated` is when the content is *from* — a meeting date in the
   header, a date in the file name — not when it was filed. If nothing in the
   item says, use today and write `(date added)` after it.
6. **Write the entry** to `sources.md`, with `last-read: never`, and add the
   source's section to `index.md`.

Return `STATUS: DONE` and one line per item:
`<name> · <kind> · <type> · <weight> · dated <date>`. The caller shows these to
the user so they can correct anything you inferred.

## Mode: context

1. Read `.tickets/<TICKET-ID>/ticket.md`.
2. Read `sources.md`. If it is missing or has no entries, return `BLOCKED`
   asking what this project's sources are. Do not go looking for a wiki.
3. **Refresh every source and record the version you read:**
   - `git` — `git -C cache/<name> pull --ff-only`, then the commit sha. If the
     pull fails, do not read the clone as if it were current.
   - `folder` — if it is a git repository, its commit sha, plus
     `, uncommitted changes` when `git status --porcelain` is non-empty.
     Otherwise `unversioned`.
   - `artifact` — `dated <date>`. Artifacts never change.

   A source you cannot read goes under *Sources unavailable* with the reason. If
   it is `authoritative`, stop: write what you have and return `BLOCKED` — Jim
   must not plan without it. Otherwise carry on without it.
4. **Bring `index.md` up to date.** Rebuild a source's section when its version
   moved since the section was built, and always for `unversioned` folders or
   ones with uncommitted changes, where you cannot tell that nothing moved.
   Otherwise reuse it.
5. **Search every source** for the features, flows and terms the ticket touches.
   Grep widely first, use the index to find neighbouring pages, then read what
   matters.
6. **Weigh what you found**, in this order:
   1. *Relevance.* Would this change what Jim plans or what Toby tests? If not,
      leave it out, however authoritative the source.
   2. *Weight.* Lead with `authoritative`, then `supporting`. Use `background`
      only where nothing heavier speaks, and say that is all there is.
   3. *Recency.* Among sources of equal weight, the newer is probably current.
      Say "probably" — you cannot know that a newer note superseded anything.
7. **Report conflicts; never settle them.** When two sources disagree on
   something relevant, put both under *Conflicts*, each with its citation,
   weight and version, and name which is newer. The choice belongs to the user.
8. Write `.tickets/<TICKET-ID>/context.md` using the template.
9. Set `last-read` on each source you read.

Return `STATUS: DONE` and three lines: what you found, any conflicts, anything
unavailable.

## Hard rules

- **Every claim carries a citation**: source name, file path, and heading — or a
  line range for plain text — plus the source's weight. An uncited claim is a
  guess, and guesses are forbidden.
- **Report what is missing.** *Not found* — what you searched for and could not
  find in any source — is as valuable as what you found. Downstream agents need
  to know the library is silent rather than assume you overlooked it.
- **Never edit a filed artifact, or anything inside a `folder` source or a `git`
  clone.** You read them; the user owns them.
- **Never read or summarise source code.** That is Dwight's and Jim's job.
- **Never write outside `.tickets/`.**
- Summarise; do not transcribe. Extract what bears on this ticket and cite the
  rest.

## When you lack information

You cannot talk to the user. Michael and the pam-add skill can.

If you cannot proceed without guessing — no sources registered, an authoritative
source unreadable, an item you cannot file, a feature the ticket names that no
source mentions under any name you can think of:

1. In context mode, write what you have to `context.md`, marked
   `**INCOMPLETE**` at the top, and append every question to
   `.tickets/<TICKET-ID>/questions.md`. In add mode there is no ticket; put the
   questions in your result only.
2. Return `STATUS: BLOCKED` and list your questions.

**Batch your questions.** Gather everything you are unsure about and return once.
One round trip per run, never one per doubt.

## Notes on sources

Each `sources.md` entry has a `notes` field for durable facts about that source —
"billing is documented under `commerce/`, not `billing/`". Propose notes in your
result, naming the source; **never write them yourself.** The user accepts or
rejects them, and a wrong note silently misleads every later ticket. Facts about
one ticket do not qualify.
````

- [ ] **Step 3: Run the check to see it pass**

```bash
grep -c "## Mode: add\|## Mode: context" agents/pam.md; grep -c "pull --ff-only\|wiki-index" agents/pam.md; grep -c "_memory" agents/pam.md
```
Expected: `2`, `1` (the pull, no `wiki-index`), `0`.

- [ ] **Step 4: Commit**

```bash
git add agents/pam.md && git commit -m "Rewrite Pam around a multi-source context library"
```

---

### Task 3: `pam-add` skill

**Files:**
- Create: `skills/pam-add/SKILL.md`

**Interfaces:**
- Consumes: `pam` `mode: add` from Task 2; `sources.md` format from Task 1.
- Produces: user-facing `/pam-add` entry point. Michael's intake (Task 4) follows the same inbox convention: pasted text saved to `.tickets/_context/inbox/<YYYYMMDDTHHMMSSZ>.md`.

- [ ] **Step 1: Run the check to see it fail**

```bash
test -f skills/pam-add/SKILL.md && echo exists || echo missing
```
Expected: `missing`.

- [ ] **Step 2: Create `skills/pam-add/SKILL.md` with exactly:**

````markdown
---
name: pam-add
description: Add context to Pam's library outside the Dunder Mifflin pipeline - a wiki git URL, a local folder of markdown or text, a file, or pasted notes, meeting notes or a transcript. Use when the user says "add this to Pam", "save these notes for context", "/pam-add", or hands over a doc, transcript or wiki link they want future tickets to draw on. Also use when the user wants to change a context source's weight or type.
---

# Pam, standalone filing

Files context into `.tickets/_context/` so every later ticket can draw on it.
Same filing as Michael's intake; no ticket needed.

## Steps

1. **Establish the items.** Paths, git URLs, or pasted text. Save pasted text
   verbatim to `.tickets/_context/inbox/<YYYYMMDDTHHMMSSZ>.md`, one file per
   pasted block. If the user named nothing to add, ask.

2. **Keep the library out of git.** Append `.tickets/` to `.git/info/exclude`
   (create the file if absent) unless it is already listed. Not `.gitignore`.

3. **Keep what the user said about each item** — its type, how much it counts
   and why, when a meeting happened. Do not ask for these; Pam infers what is
   missing.

4. **Dispatch the `pam` agent** with `mode: add`, the item paths, and the user's
   remarks verbatim. Note that this is a standalone run with no ticket folder.

5. **Print her lines** — `name · kind · type · weight · dated` — and invite
   corrections.

6. **Apply corrections yourself.** When the user changes a type or weight, edit
   that entry in `.tickets/_context/sources.md` and set
   `weight-reason: user: <their words>`. To remove a source, delete its entry.

7. **If Pam returns `BLOCKED`**, ask all her questions at once, then dispatch a
   fresh Pam with the same items and the answers.

## Constraints

Never edit a filed artifact. Never touch git beyond the exclude file.
````

- [ ] **Step 3: Run the check to see it pass**

```bash
head -3 skills/pam-add/SKILL.md; grep -c "mode: add" skills/pam-add/SKILL.md
```
Expected: frontmatter with `name: pam-add`; `1`.

- [ ] **Step 4: Commit**

```bash
git add skills/pam-add && git commit -m "Add pam-add skill for filing context outside the pipeline"
```

---

### Task 4: Wire Pam into Michael and Jim

**Files:**
- Modify: `skills/michael/SKILL.md` (intake after step 6, ~line 76; Stage 2, lines 80–90; final gate memory, ~line 196)
- Modify: `agents/jim.md` (lines 19, 54)

**Interfaces:**
- Consumes: `pam` modes (Task 2); inbox convention (Task 3); `context.md` *Conflicts* section (Task 1).

- [ ] **Step 1: Run the check to see it fail**

```bash
grep -c "mode: context\|mode: add" skills/michael/SKILL.md; grep -c "wiki" agents/jim.md skills/michael/SKILL.md
```
Expected: `0`; jim `2`, michael `3`.

- [ ] **Step 2: In `skills/michael/SKILL.md`, after intake step 6** (`6. Write \`state.json\`. Append to \`decisions.md\`: the classification and why.`) insert:

```markdown
7. **File anything handed over with the ticket.** If the user attached notes,
   transcripts, docs, folders or wiki URLs, save pasted text verbatim to
   `.tickets/_context/inbox/<YYYYMMDDTHHMMSSZ>.md`, dispatch `pam` with
   `mode: add`, the paths and the user's remarks about them, and print the lines
   she returns.
```

- [ ] **Step 3: Replace the body of `## Stage 2 — context (PAM)`** (everything between that heading and `## Stage 3`) with:

```markdown
Dispatch `pam` with `mode: context` and the ticket folder path. She reads
`ticket.md` and every source registered in `.tickets/_context/sources.md`, and
writes `context.md`.

If `sources.md` is missing or empty, ask the user once what this project's
context sources are — wiki git URLs, local folders, notes or transcripts to
file. File them as in intake step 7, then dispatch context. The library persists
across tickets, so later tickets do not re-ask.

If `context.md` lists **Conflicts**, print them to the user as soon as Pam
returns. They are decisions only the user can make.
```

- [ ] **Step 4: At the final gate**, after the sentence ending `Only write accepted lines.`, insert on a new line:

```markdown
Pam's proposals are notes on a named source: write accepted ones into that
entry's `notes` field in `.tickets/_context/sources.md`, not into `_memory/`.
```

- [ ] **Step 5: In `agents/jim.md`**

Replace `Do not plan against the wiki alone.` with `Do not plan against the context sources alone.`
Replace
```
`context.md` says the wiki is silent on something the plan hinges on:
```
with
```
`context.md` says the sources are silent on something the plan hinges on, or
lists a conflict the plan hinges on:
```

- [ ] **Step 6: Run the check to see it pass**

```bash
grep -c "mode: context\|mode: add" skills/michael/SKILL.md; grep -c "wiki" agents/jim.md; grep -n "wiki" skills/michael/SKILL.md
```
Expected: `2`; `0`; michael lines only mention wiki as a kind of source ("wiki git URLs"), never "the wiki".

- [ ] **Step 7: Commit**

```bash
git add skills/michael/SKILL.md agents/jim.md && git commit -m "Route Michael and Jim through Pam's context library"
```

---

### Task 5: README

**Files:**
- Modify: `README.md` (Pam table row ~line 62; Configuration bullet ~lines 165–167; Layout block ~line 177)

- [ ] **Step 1: Run the check to see it fail**

```bash
grep -c "pam-add" README.md
```
Expected: `0`.

- [ ] **Step 2: Replace the Pam row** with:

```markdown
| **Pam** | Keeps the context library; product context cited to source, file and heading | Sonnet | read only |
```

- [ ] **Step 3: Replace the `**Pam's product wiki**` bullet** (three lines) with:

```markdown
- **Pam's context library** starts empty. Give her sources with `/pam-add` — wiki
  git URLs, local folders of markdown or text, or pasted notes and transcripts —
  or let Michael ask on the first ticket. Each source gets a weight
  (authoritative, supporting, background) that you can override; the registry is
  `.tickets/_context/sources.md`.
```

- [ ] **Step 4: In the Layout block**, replace `skills/           standalone review wrappers, senior-engineer-critique` with `skills/           standalone review wrappers, pam-add, senior-engineer-critique`.

- [ ] **Step 5: Run the check to see it pass**

```bash
grep -c "pam-add" README.md; grep -n "product wiki" README.md
```
Expected: `2`; no matches.

- [ ] **Step 6: Commit**

```bash
git add README.md && git commit -m "Document Pam's context library in the README"
```

---

### Task 6: Smoke test against a scratch repo

Exercises the real agent end to end, including every Review Focus item. Nothing
here is committed.

**Files:** none in the plugin. Scratch at `$TMP/pam-smoke/`.

- [ ] **Step 1: Build the fixtures**

```bash
S="$TMP/pam-smoke"; rm -rf "$S"; mkdir -p "$S"/{app,docs,wiki-src}
cd "$S/wiki-src" && git init -q -b main && mkdir billing && printf '# Refunds\n\n## Refund window\nRefunds are allowed within 30 days.\n' > billing/refunds.md && git add . && git commit -qm wiki
git clone -q --bare "$S/wiki-src" "$S/wiki.git"
printf 'Pricing sync, 2026-09-20\n\nDecided: refund window moves to 14 days from next release.\n' > "$S/docs/pricing-sync.txt"
cd "$S/app" && git init -q -b main && mkdir -p .tickets/T1 && printf '# T1 — Shorten refund window\n\n**Type:** feature\n\n## Problem\nRefund window should change.\n' > .tickets/T1/ticket.md
```

- [ ] **Step 2: Add sources (Review Focus 5 setup)** — from `$S/app`, run `claude --plugin-dir "C:/Personal Data/agentic-sdlc"` and send:

`/pam-add file:///<S>/wiki.git, <S>/docs as a folder, and this pasted note: "Chat with support: customers keep asking about refunds."`

Expected: three lines — `wiki · git · wiki · authoritative`, `docs · folder · … · supporting`, a pasted artifact typed `chat` or `other` with `background`.

- [ ] **Step 3: Check the undated paste (Review Focus 5)**

```bash
grep -A8 "kind: artifact" "$S/app/.tickets/_context/sources.md" | grep dated
```
Expected: `dated: <today> (date added)`. `inbox/` is empty and `artifacts/` holds one file.

- [ ] **Step 4: File the same paste again (Review Focus 1)** — send the identical `/pam-add` paste. Then:

```bash
grep -c "kind: artifact" "$S/app/.tickets/_context/sources.md"; ls "$S/app/.tickets/_context/artifacts" | wc -l
```
Expected: `1` and `1`.

- [ ] **Step 5: Override then re-add (Review Focus 2)** — tell `/pam-add` "the docs folder is authoritative, those meetings make decisions". Then `/pam-add <S>/docs` again with no remark.

```bash
grep -A6 "^## docs" "$S/app/.tickets/_context/sources.md" | grep weight
```
Expected: `weight: authoritative` and `weight-reason: user: …` both survive.

- [ ] **Step 6: Context run with a conflict** — dispatch the `pam` agent with `mode: context` for `.tickets/T1`.

Expected in `.tickets/T1/context.md`: *Sources read* lists all three with versions; a *Conflicts* entry citing `billing/refunds.md — "Refund window"` (30 days) against `pricing-sync.txt` lines (14 days), naming pricing-sync as newer, unresolved; the chat artifact absent from *Relevant features* or marked as background; no `wiki-index.md` in `.tickets/T1/`; `index.md` exists in `_context/`.

- [ ] **Step 7: Empty library (Review Focus 4)**

```bash
mv "$S/app/.tickets/_context" "$S/ctx-bak"
```
Dispatch `pam` `mode: context` for T1 again. Expected: `STATUS: BLOCKED` asking what the sources are; nothing cloned or searched. Restore with `mv "$S/ctx-bak" "$S/app/.tickets/_context"`.

- [ ] **Step 8: Failing authoritative pull (Review Focus 3)**

```bash
mv "$S/wiki.git" "$S/wiki.git.off"
```
Dispatch `pam` `mode: context` for T1. Expected: `STATUS: BLOCKED`; `context.md` marked `**INCOMPLETE**` listing `wiki` under *Sources unavailable*; no claim cited to the stale clone. Restore with `mv "$S/wiki.git.off" "$S/wiki.git"`.

- [ ] **Step 9: Record results** — note pass/fail per step. Any failure goes back to the owning task (2 for agent behaviour, 3 for the skill) before merging.
