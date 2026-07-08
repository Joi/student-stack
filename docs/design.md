# Design

One VM per student, one vault per student, one shared notebook per class, two reporting lanes. This document explains each layer and the decisions behind it. The prototype is one machine (the instructor as student #0); the target is a 10-20 student cohort from September 2026.

## The machine

A hosted Ubuntu VM per student, provisioned from a common image by the instructor. Students do not self-install; a broken machine is destroyed and re-provisioned. Everything below is installed by one idempotent bootstrap script.

| Layer | What | Why |
|---|---|---|
| Harness | [pi.dev](https://pi.dev), pinned version | chi is Pi-native; sessions are plain JSONL |
| Sessions | [chi](https://github.com/henkaku-center/chi) | cross-machine resume (VM over SSH + laptop is one workflow, not two), consent-scoped sharing keyed to GitHub identity |
| Learning loop | [jilog](https://github.com/Joi/jilog), one nightly run | corrections, errors, recurring patterns, and spend across the student's own agent sessions, written as a digest into their vault |
| Knowledge | markdown vault (`~/vault`, git repo, private to the student) | see below |
| Reporting | nightly report + semantic updates | see [report-schema.md](report-schema.md) |

The daily loop is five small timers: create today's daily note (06:00), run jilog (23:00), build the report (23:30), sync the vault every 15 minutes, health-probe every 5.

## The vault

A minimal Obsidian-compatible vault ([vault-skeleton/](../vault-skeleton/)), distilled from a vault that has run for years at several thousand notes. The conventions that earned their keep:

- **Core plugins only.** No community-plugin dependencies; a fresh Obsidian install works. Dashboards are static pages the agent rewrites, not live queries.
- **Description-first frontmatter.** Every file carries a one-line `description` in YAML frontmatter, so agents (and humans) can filter before reading across a growing vault.
- **Capture, then promote.** `daily/` is the capture spine; durable knowledge is promoted to `notes/`; when a topic outgrows `notes/`, it graduates to its own `domains/<topic>/` folder with its own index.
- **The agent is a first-class writer.** `AGENTS.md` in the vault root tells any coding agent how to behave: edit files directly, preserve wikilinks, always write descriptions.

## Sync: git everywhere

Each vault is a private git repo. The VM agent commits and pushes on a timer; the laptop runs Obsidian with the obsidian-git plugin. Agent and human usually touch different files, so conflicts are rare, and when they happen git preserves both sides with history.

The **shared class notebook** is a second git repo, cloned into every vault at `shared/`. The convention that keeps merges trivial: each student writes their own files (`shared/sessions/<date>-<handle>.md`, `shared/people/<handle>.md`); jointly-edited pages are limited to the index and topic hubs.

Why not a real-time collaboration layer? We evaluated [Relay](https://relay.md) (excellent live co-editing UX) and [obsidian-livesync](https://github.com/vrtmrz/obsidian-livesync) (solid self-hosted per-user sync). Both fail the same test: the primary writer here is a **headless agent on a VM**, and both are Obsidian-plugin-bound with no mature headless path. Git serves both sync problems with one toolchain, gives provenance for agent writes, and the course teaches git anyway. Relay remains attractive later as a human-only live layer on the shared folder.

## Reporting: two lanes

Detailed in [report-schema.md](report-schema.md). The short version:

- **Lane A (machine lane).** A nightly cron on the VM emits one fixed-schema JSON: session counts, learning signals from jilog, spend by model, vault activity, and the student's self-authored asks/offers. Mechanical (shell + jq, no LLM), deterministic, cell-only. The same file is committed into the student's vault — the student always sees exactly what reports out.
- **Lane B (semantic lane).** Voluntary, agent-written `update` records (title, summary, key_points, source, timestamp) pushed to a per-student [Underlay](https://www.underlay.org) collection from whatever agent the student is already talking to, on any machine. Schema-optional by design: agents have gotten good at aligning schemas after the fact, so we do not over-specify up front.

The instructor's side aggregates Lane A into a class dashboard (activity, spend, staleness) and runs a matching pass over Lane B: pair asks with offers, flag topic overlap, suggest who should talk to whom.

**The privacy line, stated once.** chi handles session-level sharing with per-session consent, controlled by the student. Lane A carries coarse telemetry, disclosed in course docs and visible in the student's vault. Neither carries session content. Raw student sessions are never collected.

## Provider economics

Open question. The prototype tests running the harness against the hosting provider's LLM proxy, which would put zero provider API keys on student disks. The alternative is one scoped, revocable key per machine. Cost per student per semester is a number this prototype exists to measure — jilog's spend section reports observed cost per session, per model.

## Status and sequence

- **July 2026** — prototype machine live, instructor dogfoods as student #0.
- **August 2026** — full dry run: provision, onboard, build a small project end to end. Go/no-go for each optional layer.
- **September 2026** — first cohort.
