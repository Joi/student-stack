---
description: Rules for the coding agent that reads and writes this vault. Read this before editing any file.
---

# Agent rules for this vault

This vault is a markdown atlas. You (the coding agent) are its primary writer.
A human reads it in Obsidian after it syncs to their laptop. Write for that reader.

## How to edit

- Edit markdown files directly. Do not wrap edits in code fences or emit diffs.
- Preserve `[[wikilinks]]`. When you rename a note, update the links that point to it.
- Every file starts with YAML frontmatter that includes a `description:` field of at
  least 30 characters. The description says what the file is; it is what makes the
  vault searchable and what the nightly report scrapes for project summaries.
- Write in plain, direct prose. No marketing language, no emoji.

## Where things go

- Capture first in `daily/`. Each day has one note. Put raw notes, decisions,
  questions, and links there as they happen.
- Promote durable knowledge out of `daily/` into `notes/` once it is worth keeping.
  A daily note is a log; a note in `notes/` is a fact or a method you will reuse.
- One person per file in `people/`, named `Firstname Lastname.md`.
- Notes in `notes/` use kebab-case filenames (e.g. `webhook-signatures.md`).
- Projects go in `projects/`, one file each, with `status:` and `description:` in
  the frontmatter. The nightly report reads those two fields.
- Reference material and transcripts go in `references/`. Attachments go in
  `attachments/`.
- When a topic outgrows a single note in `notes/`, graduate it to `domains/<topic>/`
  and split it into multiple files there. Leave a stub in `notes/` that links to the
  new domain.

## Sharing is the student's decision

Class sharing (`student-update enable-sharing`) is a consent step held by the human.
Never enable or disable it yourself, and never move content into `shared/` that the
student did not ask to share. If sharing something would help, say so and let the
student decide.

## What not to touch

- `learning/digests/` is written by the nightly learning job. Read it, do not edit it.
- `report/student-report.json` is generated. Read it, do not edit it.
- `shared/` is a separate shared repository (the class notebook). See `_STRUCTURE.md`.

See [[_STRUCTURE]] for the full routing table and naming rules.
