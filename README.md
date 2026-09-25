# agentic-sdlc

A multi-agent software development pipeline for [Claude Code](https://claude.com/claude-code).
Hand it a ticket; get back a planned, tested, twice-reviewed working tree.

> ### 📎 [Read the design record →](https://claude.ai/code/artifact/c0bc2328-7651-45bb-b44d-1523ac0dded3)
> The full architecture, every decision, and what was rejected — with diagrams.
> Start there if you want the *why*.

## In practice

Roughly **30+ tickets** over **four months** of use — five counting development.
Mostly bugs and features in fairly even measure; spikes are the smallest slice,
which is why the spike route is also the simplest.

Structurally, and checkable by reading the repo:

| | |
|---|---|
| Agents | 8 — one orchestrator, seven subagents |
| Ticket routes | 3 — spike, bug, feature |
| Human gates | 2 — the plan, and the finished work |
| Agents that can write production code | 1 |
| Reviewers with any write tool | 0 |
| Loop ceilings | 2 plan reviews · 3 TDD cycles · 2 code reviews |
| Artifacts written per ticket | 9 |

## What this is

Handing an entire ticket to one agent fails in a predictable way. It plans and
implements in the same breath, so nothing ever challenges the plan. It reviews its
own code, so the review is theatre. And it fills gaps by guessing, because asking
is harder than assuming.

`agentic-sdlc` splits that work across eight agents with separate jobs, separate
contexts, and separate permissions. One plans. A different one attacks the plan. A
third writes tests before any code exists. Only one of them can edit your source.
Two more review the result, and neither of them can touch it.

They are named after characters from *The Office*. Not for the joke — casting makes
responsibilities hard to blur. Angela is hostile to every proposal. Oscar counts.
Kevin makes the chili.

## Pipeline

```
                                          ┌── spike ──→ brainstorm ──→ done (no code)
                                          │
intake ──→ context ──→ [ repro ] ──→ plan ─┴─→ plan-review ──→ ▓ GATE ▓
MICHAEL      PAM        DWIGHT       JIM        ANGELA          you approve
                     bug only,                  max 2 rounds
                  consent-gated

──→ tests ──→ implement ──⇄──→ code-review ──→ ▓ GATE ▓ ──→ done
     TOBY        KEVIN            ANGELA ∥ OSCAR   you approve   uncommitted
             max 3 TDD cycles     max 2 rounds                   working tree
```

| Agent | Role | Model | Source access |
|---|---|---|---|
| **Michael** | Orchestrator. The only agent that talks to you. | your session | delegates |
| **Pam** | Keeps the context library; product context cited to source, file and heading | Sonnet | read only |
| **Dwight** | Reproduction, evidence, affected code | Sonnet | read only |
| **Jim** | Planning; approaches considered and rejected | Opus | read only |
| **Angela** | Architecture, necessity, regressions vs. the design | Opus | **no write tools** |
| **Toby** | Tests first, mapped to acceptance criteria | Sonnet | tests only |
| **Kevin** | Implementation | Sonnet | **sole writer** |
| **Oscar** | Line-by-line critique | Opus | **no write tools** |

Opus sits in the three seats where being wrong is most expensive — planning and the
two review lenses. Everything downstream of a good plan runs on Sonnet.

## What it does differently

- **Permissions do the enforcing.** Reviewers are declared with no write tools at
  all, so "reviewers never edit code" is a property of the harness, not a rule a
  model might talk itself out of.
- **No agent guesses.** Missing information stops the stage and reaches you as a
  question. Nothing proceeds on an assumption.
- **Nothing accumulates.** Every stage runs in a fresh context, fed by files rather
  than by conversation history.
- **Every loop has a ceiling.** Hitting one is a stop that hands you the evidence,
  never a silent pass.
- **Nothing touches git.** The deliverable is an uncommitted working tree. Committing
  stays yours.
- **You hold two gates** — the plan, and the finished work. Everything between runs
  unattended.

Each of these is argued properly in the [design record](https://claude.ai/code/artifact/c0bc2328-7651-45bb-b44d-1523ac0dded3).

## How it grew

Nothing here was designed up front. Each stage was added because the version before
it failed in a specific, repeated way:

1. **Code review** — the original tool, and still the highest-value single step.
2. **Architecture review** — because reviewing code says nothing about whether the
   approach was right, or needed at all.
3. **Code writing** — once the plan was being challenged properly, implementing
   against it was the obvious next step.
4. **TDD** — because code written without tests first drifts toward whatever is
   easiest to make pass.
5. **Context fetching** — because everything above was quietly guessing at product
   behaviour that was already written down in the wiki.

## What was hard to get right

The four things that took the most iteration, and what each one is actually fighting:

- **The BLOCKED protocol.** Subagents can't prompt the user, so "ask, don't assume"
  had to stop being an instruction and become plumbing. An agent that lacks
  information writes its partial work, files its questions, and returns `BLOCKED`.
  Making that more attractive than guessing was the single hardest part.
- **Angela's prompt.** An adversarial reviewer oscillates between rubber-stamping
  and manufacturing findings to look thorough. Both are useless. Getting her to
  approve sound work *and* reject unsound work took more attempts than anything else.
- **Hard iteration caps.** Without a ceiling, a fix-and-review loop will happily run
  forever. The caps are deliberately low, and reaching one is a stop that surfaces
  evidence rather than a silent pass.
- **A fresh agent every invocation.** Reusing a context is tempting and always wrong
  — reviewers start confusing one run with another. Every re-run is a new agent, fed
  by files.

## Ticket state

Every stage reads and writes files in the repo you are working on, ignored via
`.git/info/exclude` so nothing reaches a pull request. The result is a readable trail
of what was decided and *why* — including `plan.md` with the approaches that lost,
and `decisions.md`, the file you read six weeks later when someone asks why the fix
looks like that.

```
.tickets/PROJ-123/
  state.json  ticket.md  context.md  repro.md  plan.md  tests.md
  reviews/    questions.md  decisions.md  worklog.md
```

## Install

```bash
/plugin marketplace add /path/to/agentic-sdlc
/plugin install dunder-mifflin@agentic-sdlc
```

## Use

```
/michael PROJ-123            # Jira ID, GitHub issue URL, file path, or pasted text
/michael resume PROJ-123     # pick up where it left off
/michael status PROJ-123     # print stage and gates, change nothing

/angela-review               # adversarial design review, standalone
/oscar-review                # line-by-line code critique, standalone
```

Michael classifies the ticket as a **spike** (no code — terminates at brainstorming),
a **bug** (adds a consent-gated reproduction stage), or a **feature** (straight to
planning), and routes accordingly.

## Configuration

Two integrations need setting up:

- **Jira intake** needs an MCP server name. Pasted text and GitHub issue URLs work today.
- **Pam's context library** starts empty. Give her sources with `/pam-add` — wiki
  git URLs, local folders of markdown or text, or pasted notes and transcripts —
  or let Michael ask on the first ticket. Each source gets a weight
  (authoritative, supporting, background) that you can override; the registry is
  `.tickets/_context/sources.md`.

## Layout

```
.claude-plugin/   plugin and marketplace manifests
agents/           seven subagent definitions
skills/michael/   orchestrator, state schema, artifact templates
skills/           standalone review wrappers, pam-add, senior-engineer-critique
```

## License

MIT
