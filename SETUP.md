# Setting up shared context in your repo

A 5-minute setup. Every Claude Code session in your repo will start by
fetching the company's foundational `CLAUDE.md` from
[`fusioncontext`](https://github.com/nakesekander-cpo/fusioncontext) and
inlining it into the model's context. You never copy or paste the content
— one PR against `fusioncontext` updates every team's next session.

## Prerequisites

- A repo with a Claude Code session running against it (Claude Code on
  the web or the local CLI).
- Outbound HTTPS to `raw.githubusercontent.com`. This is allowed under
  the default network policy on Claude Code on the web. If your
  environment uses a stricter policy, see
  [Claude Code on the web docs](https://code.claude.com/docs/en/claude-code-on-the-web).
- 4 small changes to your repo, all under `.claude/` and `.gitignore`.

## Steps

### 1. Add the fetch script

Create `.claude/hooks/session-start.sh` and make it executable
(`chmod +x .claude/hooks/session-start.sh`):

```bash
#!/usr/bin/env bash
set -euo pipefail

DEST_DIR="${CLAUDE_PROJECT_DIR:-.}/.claude/context"
DEST_FILE="$DEST_DIR/foundational.md"
SOURCE_URL="https://raw.githubusercontent.com/nakesekander-cpo/fusioncontext/main/CLAUDE.md"

mkdir -p "$DEST_DIR"

if ! curl -fsSL "$SOURCE_URL" -o "$DEST_FILE"; then
  echo "warning: failed to fetch foundational context from $SOURCE_URL; using last cached copy if present" >&2
fi
```

The `if ! curl …` block keeps the session running even if the fetch
fails — Claude will fall back to whatever copy is already on disk (or
none, on first run).

### 2. Register the hook

Add a `SessionStart` block to `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

**If your repo already has a `hooks` block** (e.g. a `Stop` hook), merge
the `SessionStart` array into the existing `hooks` object — don't
replace it. Example with both:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-start.sh" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "your-existing-stop-command" }
        ]
      }
    ]
  }
}
```

### 3. Reference the fetched file from your `CLAUDE.md`

Add a single line near the top of your repo's `CLAUDE.md` so Claude
inlines the foundational context on every turn:

```
@.claude/context/foundational.md
```

If your repo doesn't have a `CLAUDE.md` yet, create one with just that
line plus whatever repo-specific guidance you want below it.

### 4. Gitignore the fetched copy

`.claude/context/foundational.md` is a regenerated artifact — committing
it re-introduces the drift this setup is designed to prevent. Add this
line to `.gitignore`:

```
.claude/context/
```

Commit the script, the `settings.json` change, the `CLAUDE.md`
reference, and the `.gitignore` line. Do **not** commit
`.claude/context/`.

## Verify it works

1. Start a fresh Claude Code session in your repo.
2. Confirm the file appears on disk: `ls -la .claude/context/foundational.md`.
3. Ask Claude: *"Summarize the foundational context loaded from
   fusioncontext."* The answer should mirror the sections of
   [`fusioncontext/CLAUDE.md`](https://github.com/nakesekander-cpo/fusioncontext/blob/main/CLAUDE.md)
   (Mission, Product, How we think about the work, Glossary).

If that works, you're done. Every future session in this repo will
pull the latest foundational context automatically.

## Updating foundational context

When the foundational context needs to change (new product, updated
principle, new term in the glossary), open a PR against `fusioncontext`
editing
[`CLAUDE.md`](https://github.com/nakesekander-cpo/fusioncontext/blob/main/CLAUDE.md).
Merge to `main`. Every team's next Claude session picks it up — nothing
to re-install, nothing to broadcast.

## Troubleshooting

**Hook didn't run / file isn't appearing on disk.**
Check `.claude/settings.json` is valid JSON (a missing comma will
silently disable hooks). Confirm the script is executable
(`ls -l .claude/hooks/session-start.sh` should show `x` bits). Run the
script manually to see its output: `./.claude/hooks/session-start.sh`.

**File is on disk but Claude doesn't seem to know about it.**
The `@.claude/context/foundational.md` line in your repo's `CLAUDE.md`
is missing or mis-typed. The `@` prefix is required — it's what tells
Claude to inline the file.

**Script reports a 404 from `raw.githubusercontent.com`.**
The file isn't on `main` yet in `fusioncontext`. Until the initial
content is merged from `claude/shared-context-space-er3gi` to `main`,
the raw URL will 404. Ping the owner of the `fusioncontext` repo.

**Script fails with `curl: command not found` or DNS errors.**
Your environment's network policy is blocking the fetch. See the
[Claude Code on the web docs](https://code.claude.com/docs/en/claude-code-on-the-web)
for the policies that allow outbound HTTPS.

## What NOT to do

- **Don't `git clone` `fusioncontext` into your repo.** A clone is a
  point-in-time snapshot that immediately starts drifting. The hook
  fetches one file fresh each session — that's the whole point.
- **Don't commit `.claude/context/`.** A committed copy will shadow the
  fresh fetch and reintroduce drift.
- **Don't paste foundational content into your repo's `CLAUDE.md`.**
  Use the `@` reference instead. Inline copies become stale and start
  contradicting `fusioncontext`.
