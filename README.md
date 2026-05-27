# fusioncontext

Shared, always-fresh context for every product team's Claude session.

**Teams onboarding for the first time:** see [SETUP.md](./SETUP.md) for
the full step-by-step guide.

**Single source of truth.** Foundational context (mission, product
overview, glossary, principles) lives in `CLAUDE.md` here. Consumer
repositories pull this file at the start of each Claude session, so an
edit on `main` propagates to every team without anyone having to
re-install anything.

## How a consumer repository uses this

Add a `SessionStart` hook to the consumer repo's
`.claude/settings.json` that fetches `CLAUDE.md` from this repository
into the session before Claude reads its context.

Minimal example — drop into the consumer repo's `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "curl -fsSL https://raw.githubusercontent.com/nakesekander-cpo/fusioncontext/main/CLAUDE.md -o .claude/context/foundational.md"
          }
        ]
      }
    ]
  }
}
```

Then reference `.claude/context/foundational.md` from the consumer
repository's own `CLAUDE.md` (e.g. `@.claude/context/foundational.md`)
so Claude loads it on every turn.

For private repos, swap `curl` for an authenticated `gh api` call or a
shallow `git clone --depth=1`.

## Updating foundational context

1. Open a PR against `fusioncontext` editing `CLAUDE.md`.
2. Merge to `main`.
3. Every team's next Claude session picks it up automatically — no
   action required on their side.

## What does NOT belong here

- Repo-specific build, test, or deployment instructions — those go in
  the consumer repository.
- Per-team sprint state or task assignments — those go in Paperclip.
- Secrets or credentials.
