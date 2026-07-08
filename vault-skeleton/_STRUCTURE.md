---
description: The routing table, naming rules, and growth rules for this vault. The contribution guide.
---

# Structure

How this vault is organized and how to keep it organized.

## Routing table

| Put this | Here | Named |
|---|---|---|
| Today's log, raw capture | `daily/` | `YYYY-MM-DD.md` |
| A durable note, fact, or method | `notes/` | kebab-case, e.g. `webhook-signatures.md` |
| A person | `people/` | `Firstname Lastname.md` |
| A project | `projects/` | short kebab-case, e.g. `line-bot.md` |
| Source material, transcripts, papers | `references/` | free-form |
| Images and files | `attachments/` | free-form |
| Retired notes | `_archive/` | keep the original name |
| A topic that grew past one note | `domains/<topic>/` | one dir per topic |

Generated, do not hand-edit: `learning/digests/`, `report/student-report.json`.
Separate repository: `shared/` (the class notebook).

## The description rule

Every file begins with YAML frontmatter that has a `description:` of at least 30
characters. The description is a full sentence saying what the file is. It powers
search and is the field the nightly report reads to summarize your projects, so
keep project descriptions honest and current.

## Naming

- Notes: kebab-case (`git-rebase-recovery.md`), no dates in the name.
- People: `Firstname Lastname.md`, exactly.
- Projects: short kebab-case matching how you refer to the project out loud.
- Daily notes: `YYYY-MM-DD.md`, created for you at 06:00.

## The capture-then-promote spine

Write into `daily/` as things happen. That is cheap and lossless. Later, pull the
parts worth keeping into `notes/`, `people/`, or `projects/`. The daily note stays
as the log of when and why; the promoted note is the reusable version.

## The domains growth rule

`notes/` holds one file per topic. When a topic accumulates enough that one file is
unwieldy — several distinct sub-topics, long reference sections — graduate it to
`domains/<topic>/` and split it into multiple files. Leave a short stub in `notes/`
that links to `domains/<topic>/`, so existing links keep working.
