# The class brain: Lane B, implemented

*Status: running (July 2026). This implements the semantic lane sketched in
[report-schema.md](report-schema.md) — what changed from that sketch is noted at
the end.*

Each student's machine can now share a nightly **semantic update** — what they
worked on, what they learned, what they're stuck on — with a small class server
that aggregates the class, suggests who should talk to whom, and serves a
dashboard plus an API their agents can query. Sharing is consent-gated,
transparent, and coarse by construction.

## The record: `student-update/v0`

One JSON record per student per night, built by a local job from deliberate
surfaces only:

```json
{
  "schema": "student-update/v0",
  "id": "sha256:<content address>",
  "student": {"github": "handle"},
  "date": "2026-09-15",
  "visibility": "class",
  "title": "Got LINE webhook signatures verifying",
  "summary": "2-4 sentences: what moved, what was learned.",
  "key_points": ["hmac is base64, not hex"],
  "topics": ["line-bot", "webhooks"],
  "projects": [{"name": "line-bot", "one_liner": "...", "status": "active"}],
  "asks": ["stuck on reply-token expiry"],
  "offers": ["got signature verification working"],
  "instructor_only": {"learning": {"corrections": 1, "errors": 4}},
  "provenance": {"digest": "...", "chi_nodes": ["..."], "producer": {"instruction_sha256": "..."}}
}
```

Records are content-addressed (sha256 over canonical JSON — RFC 8785 subset,
no floats), size-capped (8KB per field, 64KB per record, enforced at both ends),
and deliberately Underlay-shaped: versioned, immutable once pushed, with
row-level visibility — so exporting a student's collection to a real
[Underlay](https://www.underlay.org) collection later is a serializer, not a
redesign.

## What the producer reads — and what it never reads

The nightly job's inputs are restricted to deliberate/coarse surfaces:

- learning-signal **counts** and digest **section titles** from the nightly
  learning run (never digest bodies),
- `projects/*.md` **frontmatter** (skipping any file marked `share: false`),
- `collab.md` asks/offers (a file the student writes for sharing),
- memory-node **titles and summaries** when the session-memory layer is present
  (already sanitized and secret-scanned upstream),
- file counts.

Never: session content, vault diffs, daily notes, references. One LLM call
turns those inputs into the title/summary/topics; the instruction text is
hashed into the record's provenance.

## Consent and transparency

- The producer runs from day one in **local-only mode**: every record is
  written into the student's own vault (`report/updates/`) so they can see
  exactly what a shared record would contain — before anything is shared.
- Nothing is pushed until the student runs `student-update enable-sharing`
  once. `disable-sharing` stops future pushes. Both are logged into the
  student's vault (`report/sharing-log.md`).
- Any pushed record can be deleted from the server with the student's own
  token.
- The `instructor_only` block (struggle-signal counts, spend) travels inside
  the record — the student sees everything that leaves — and is stripped by
  the server for every class-facing view.

## The server

A small service (FastAPI + SQLite) owned by the class:

- **Auth**: per-student write tokens (a token can only write its own student's
  collection), one class read token for agents and the dashboard, and an
  instructor token that is never on student machines.
- **Dashboard**: latest update per student, topic clusters, live asks/offers,
  days-quiet, current intro suggestions, and a read-only index of the shared
  class notebook.
- **Agent API**: `GET /v0/students | /v0/updates | /v0/matches | /v0/search` —
  so a student's agent can ask "who else is stuck on webhook signatures?".
- **Matching pass** (nightly): pairs asks with offers and clusters topic
  overlap over a bounded corpus (per-student excerpt budgets; anything dropped
  is logged, never silently truncated). Output is **suggestions to humans with
  stated reasons — never automated actions.** Weekly, the suggestions are
  committed as a markdown note into the shared class notebook, which already
  syncs into every student's vault.
- **Durability**: because every pushed record also lives in its student's
  vault, the server database is fully reconstructible by re-ingesting those
  transparency copies.

## What changed from the report-schema.md sketch

- Lane B's "tokenized Underlay links" became a self-hosted collection server
  (the hourly-token expiry made unattended pushes impossible for now); the
  record format stays Underlay-compatible so that path remains open.
- The nightly VM job fires the Lane B update itself (the "once unattended auth
  is settled" caveat is settled by per-student scoped tokens).
- A consent enrollment step was added after independent review: the original
  sketch's "voluntary, agent-pushed" property is preserved by requiring an
  explicit one-time `enable-sharing` before any push, with the always-on
  local transparency copy as the preview.
