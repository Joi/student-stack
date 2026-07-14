# Design

One VM per student, one vault per student, one shared notebook per class, two reporting lanes. This document explains each layer and the decisions behind it. The prototype is one machine (the instructor as student #0); the target is a 10-20 student cohort from September 2026.

## The machine

A hosted Ubuntu VM per student, provisioned from a common image by the instructor. Students do not self-install; a broken machine is destroyed and re-provisioned. Everything below is installed by one idempotent bootstrap script.

**What this repo carries, and what it does not.** The provisioning implementation — the bootstrap script, the systemd timers, and the health probes described here — lives in the instructor's private fleet repository, not this one. This repo carries the design (this document), the report schemas ([report-schema.md](report-schema.md)), and the vault skeleton ([vault-skeleton/](../vault-skeleton/)). Publishing the bootstrap is future work.

| Layer | What | Why |
|---|---|---|
| Harness | [pi.dev](https://pi.dev), pinned version | chi is Pi-native; sessions are plain JSONL |
| Sessions | [chi](https://github.com/henkaku-center/chi) | cross-machine resume (VM over SSH + laptop is one workflow, not two), consent-scoped sharing keyed to GitHub identity; see [Sessions: chi](#sessions-chi) |
| Learning loop | [jilog](https://github.com/Joi/jilog), one nightly run | corrections, errors, recurring patterns, and spend across the student's own agent sessions, written as a digest into their vault |
| Knowledge | markdown vault (`~/vault`, git repo, private to the student) | see below |
| Reporting | nightly report + semantic updates | see [report-schema.md](report-schema.md) |

The daily loop is five small timers: create today's daily note (06:00), run jilog (23:00), build the report (23:30), sync the vault every 15 minutes, health-probe every 5.

## Sessions: chi

chi is how a session becomes a durable artifact the student keeps, rather than a chat log trapped in one provider's local storage. It wraps Pi — students run `chi`, not `pi` — and the integration has three parts: how it gets on the machine, what it does around a session, and what the student controls.

**Installation.** The bootstrap pins chi to an exact commit and runs chi's own installer, which brings Pi, the provider CLIs (Codex, Claude Code), and the secret scanners chi requires (gitleaks, trufflehog). chi is a private repository and cells hold no GitHub credentials, so each cell's clone is pre-staged with no origin remote; updates arrive as operator-shipped git bundles, and a cell can never pull chi on its own. Two properties follow. chi is additive — the machine works with chi disabled, so a broken chi never takes down the cell. And version drift is visible — the health probe compares the checkout against the pin. One policy question stays open: `chi update` works on the cell and will drift past the pin, and whether student cells should be pin-exempt for chi or drift treated as signal is deliberately unresolved until the August dry run.

**Around each session.** Four things, all local-first:

- *Provenance.* After any agent turn that changed the worktree, chi writes a scanner-checked snapshot commit on a hidden ref (`refs/chi/provenance/<branch>`), with trailers naming the session and entry that produced it. `chi blame <file>` maps a line back to the turn that wrote it. Normal branches and GitHub history stay clean.
- *Cross-machine resume.* Sessions are Pi JSONL; sanitized records sync to the chi backend, keyed to GitHub identity. `chi resume` hydrates a session on the laptop that started on the VM, or the reverse — the same workflow the Sessions row above promises.
- *Memory.* `/memory` and `/summarize` maintain level-of-detail rollups (raw turns → summaries → repo-level memory), so long-running work resumes from compressed context instead of replaying everything.
- *Workspace guard.* A session launched for one repo cannot read or edit another checkout: file tools and shell commands are blocked at the workspace boundary, with no local bypass. Cross-repo work goes through chi's own repo switching, so provenance always lands on the repo that was actually edited. On a teaching fleet this matters — a student's agent cannot wander out of the workspace it was launched in.

**What the student controls.** Signing in is an at-machine step (`chi login --use-gh`, riding the GitHub CLI's auth) — no GitHub credential ships on the image, and provider access is the separate question covered under [Provider economics](#provider-economics). Session sharing is per-session consent, held by the student; that boundary is stated once in [Reporting](#reporting-two-lanes) and chi enforces it.

chi and jilog are deliberately parallel today: two independent consumers of the same Pi JSONL, one for memory and provenance, one for learning signals. Whether they should know about each other — digests linking to provenance commits, nightly signals feeding session memory — is an open exploration ([#5](https://github.com/Joi/student-stack/issues/5)).

## The vault

A minimal Obsidian-compatible vault ([vault-skeleton/](../vault-skeleton/)), distilled from a vault that has run for years at several thousand notes. The conventions that earned their keep:

- **Core plugins only.** Nothing in the vault depends on a community plugin — a fresh Obsidian install opens it fully. Dashboards are static pages the agent rewrites, not live queries. (The laptop uses one community plugin, obsidian-git, purely as a sync convenience; git itself works from the CLI without it — see [Sync](#sync-git-everywhere) below.)
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
