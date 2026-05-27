# Foundational Context

This file is the canonical, always-loaded context for every team's Claude
session at this company. It is fetched fresh each session from the
`fusioncontext` repository, so editing it here updates every consumer the
next time they start a session.

If a section below contains `[fill in]`, replace it with real content. Keep
sentences short and factual — this file is read on every turn.

---

## Mission

[fill in: one sentence on what the company exists to do]

## Product

[fill in: 2–3 sentences naming each product line and what each does for
which user]

## How we think about the work

- [fill in: principle 1 — e.g. "ship the smallest correct change"]
- [fill in: principle 2]
- [fill in: principle 3]

## Glossary

Terms that mean something specific here and might confuse a new reader or
a model that has not seen them before.

- **[term]** — [definition]
- **[term]** — [definition]

## Source of truth

This file lives at
`https://github.com/nakesekander-cpo/fusioncontext/blob/main/CLAUDE.md`.
To change foundational context for every team, open a PR against
`fusioncontext`. Do not fork or duplicate this content into consumer
repositories — it will drift.

## Out of scope for this file

- Repo-specific build/test instructions — those belong in the consumer
  repository's own `CLAUDE.md` or `AGENTS.md`.
- Per-team workflows, sprint state, or task assignments — those belong in
  Paperclip or the team's tracker.
- Secrets, credentials, customer data.
