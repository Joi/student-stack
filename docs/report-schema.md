# Reporting: two lanes

Students' machines report out; students' sessions do not. Two lanes with different jobs:

| | Lane A — machine | Lane B — semantic |
|---|---|---|
| What | fixed-schema telemetry JSON | freeform `update` records |
| Writer | cron on the VM (shell + jq, no LLM) | whatever agent the student is talking to |
| Where from | the VM only | any machine |
| Cadence | nightly | whenever the student pushes one |
| Schema | fixed (`student-report/v0`) | schema-optional (default `update` type) |
| Consumer | class dashboard, health | matching pass: who should talk to whom |

## Lane A: `student-report/v0`

One JSON object per machine per night. Coarse by construction: counts, learning signals, and two fields the student writes deliberately for sharing — the `one_liner` on each project and the asks/offers in `collab.md`. No session content and no vault content beyond those deliberate fields, each visible in the student's own vault before it leaves. The same file is committed into the student's vault at `report/student-report.json`, so the student always sees exactly what reports out.

```json
{
  "schema": "student-report/v0",
  "cell": "cell-student0",
  "student": { "github": "handle" },
  "generated_at": "2026-07-08T23:30:00Z",
  "window_days": 1,
  "sessions": { "count": 3, "harnesses": { "pi": 3 } },
  "learning": { "corrections": 1, "errors": 4, "workarounds": 0, "patterns": 0, "p0": 0 },
  "spend": { "total_usd": "1.85", "by_model": { "claude-sonnet-5": "1.85" } },
  "vault": { "notes_total": 42, "notes_added": 5, "last_push_age_h": 2 },
  "projects": [ { "name": "line-bot", "one_liner": "LINE bot that answers from my notes", "status": "active" } ],
  "collab": { "asks": ["stuck on webhook signatures"], "offers": ["got LINE reply tokens working"] }
}
```

Sources, all local and mechanical:

- `sessions`, `learning`, `spend` — the machine-readable output of the nightly [jilog](https://github.com/Joi/jilog) run.
- `vault` — `git log` and file counts on the vault.
- `projects` — scraped from the frontmatter of `projects/*.md` (the description-first rule makes this possible without an LLM).
- `collab` — parsed from `collab.md` in the vault, a file the student writes (with their agent's help): what they are stuck on, what they figured out that others might want.

## Lane B: semantic updates

Lane A sees only the VM and has no semantics. Lane B is the "what am I actually working on" layer, using [Underlay](https://www.underlay.org) collections (content-addressed, versioned JSON records with row/column-level privacy flags):

- Each student has a collection; updates use the default `update` type: `title`, `summary`, `key_points`, `source`, `timestamp`.
- An update is pushed by handing an agent a tokenized link that carries the schema, examples, and the push protocol — from a laptop chat session, a coding session on the VM, or a cron job.
- Deliberately schema-optional: push what you have, formalize later. Aligning schemas after the fact is something agents are now good at; a rigid class schema up front is the mistake the fixed lane already covers.

The VM's nightly job can also fire a Lane B update ("summarize today from the digest and the vault diff") once unattended auth is settled — the current tokenized links expire hourly, which suits interactive pushes but not cron.

## The matching pass

The instructor's side aggregates Lane A for a class dashboard (per-student activity, spend, staleness — operational, boring on purpose) and periodically runs a matching pass over Lane B and the `collab` fields: pair asks with offers, cluster topic overlap, and suggest introductions. The output is suggestions to humans, not automated actions.
