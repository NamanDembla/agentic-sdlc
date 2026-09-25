# state.json

One per ticket, at `.tickets/<TICKET-ID>/state.json`. This file is what makes
`/michael resume` possible. Rewrite it after every stage transition.

```json
{
  "ticket": "PROJ-123",
  "type": "bug",
  "source": "jira:PROJ-123",
  "stage": "plan-review",
  "iterations": {
    "plan_review": 1,
    "tdd_loop": 0,
    "code_review": 0
  },
  "gates": {
    "repro_consent": "granted",
    "plan_approved": null,
    "final_approved": null
  },
  "updated": "2026-09-02T14:22:10Z"
}
```

## Fields

| Field | Values |
|---|---|
| `ticket` | The ticket ID. Also the folder name. |
| `type` | `spike` \| `bug` \| `feature` |
| `source` | `jira:<ID>` \| `github:<url>` \| `file:<path>` \| `pasted` |
| `stage` | see below |
| `iterations` | counters; caps are 2 / 3 / 2 |
| `gates` | `null` (not reached) \| `granted` \| `skipped` \| `approved` \| `rejected` |
| `updated` | ISO 8601 UTC |

## Legal stages

```
intake        Michael is parsing the ticket
context       Pam is gathering context from the library
repro         Dwight is reproducing (bug only)
brainstorm    spike route only; terminates
plan          Jim is planning
plan-review   Angela is reviewing the plan
gate-plan     waiting on the user
tests         Toby is writing tests
implement     Kevin is implementing
code-review   Angela and Oscar are reviewing the diff
gate-final    waiting on the user
done          finished; working tree uncommitted
blocked       a subagent returned BLOCKED; questions await the user
```

When `stage` is `blocked`, `state.json` also carries `blocked_from`, the stage to
return to once the user answers.

## Caps

| Counter | Cap | On reaching it |
|---|---|---|
| `plan_review` | 2 | carry both reviews to the plan gate; user decides |
| `tdd_loop` | 3 | stop; show failing output, Kevin's attempts, Toby's intent |
| `code_review` | 2 | carry all findings to the final gate; user decides |

A cap is a stop, never a silent pass. Never mark a stage complete because its cap
was reached.
