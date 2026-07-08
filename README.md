# student-stack

A reference stack for students building a personal AI system over one semester: a hosted agent machine, a markdown knowledge base, a nightly learning loop, and a consent-first way to share what you are working on.

Built for a 10-20 student seminar starting September 2026 at Henkaku Center. **Status: alpha — one prototype machine, no students yet.**

## What a student gets

- **A hosted Linux VM** — their machine for the semester, provisioned by the instructor from a common image. A broken machine is destroyed and re-provisioned, not debugged.
- **[pi.dev](https://pi.dev) as the coding harness**, with **[chi](https://github.com/henkaku-center/chi)** for session durability, cross-machine resume, and consent-scoped session sharing.
- **[jilog](https://github.com/Joi/jilog)** running nightly: what did my agent get wrong this week, what patterns keep recurring, what did it cost. The digest lands in the student's own vault.
- **A markdown vault** ([vault-skeleton/](vault-skeleton/)) — Obsidian-compatible, core plugins only, synced between the VM and the student's laptop with git. The agent writes it; the student reads and edits it anywhere.
- **A shared class notebook** — one git repo cloned into every vault. Each student writes their own files; agents can read and write it too.
- **Two-lane reporting** ([docs/report-schema.md](docs/report-schema.md)): a nightly machine report from the VM (sessions, spend, learning signals — no content), and voluntary semantic updates the student's agent pushes from wherever they are.

## Principles

- **Their sessions are theirs.** The instructor never has raw access to student sessions. Session sharing goes through chi's per-session consent; the machine report carries counts and topics, never content.
- **Transparency by construction.** The exact JSON that reports out is committed into the student's own vault. Nothing is sent that the student cannot read.
- **Everything is additive.** The machine works with chi disabled, with reporting disabled, with the shared notebook ignored. Each layer earns its place or gets turned off.
- **Plain files.** Markdown, JSONL, git. Any future agent or tool can read the whole system without an API.

## Design

- [docs/design.md](docs/design.md) — the stack, layer by layer, and the reasoning behind the choices (including why the shared notebook is git and not a real-time CRDT layer, for now).
- [docs/report-schema.md](docs/report-schema.md) — the `student-report/v0` schema and the semantic update lane.

## License

MIT
