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
