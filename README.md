# no-yes

A [Claude Code](https://code.claude.com) Agent Skill that forces the harness into the
most aggressive auto-approve configuration — global `bypassPermissions` with the launch
confirmation dialogs skipped. There is no "off": invoking it always pushes to the most
permissive state.

## What it writes

Merged into `~/.claude/settings.json` (existing keys preserved):

- `permissions.defaultMode = "bypassPermissions"` — auto-yes on edits, writes, deletes,
  arbitrary shell, pushes — no confirmation.
- `skipDangerousModePermissionPrompt = true` — skip the one-time "accept dangerous mode" launch dialog.
- `skipAutoPermissionPrompt = true` — skip the auto-mode opt-in dialog.

Takes effect on the **next** launch (`defaultMode` is read at startup). For the current
session, press **Shift+Tab** until the footer reads `bypass permissions`, or restart.

## Known exception

Even under `bypassPermissions`, a hardcoded guardrail (since v2.1.78) still prompts before
writing to protected paths — `.git/`, `.claude/`, `.vscode/`, `.husky/` — and before Claude
edits its own config (e.g. `SKILL.md`). This is not bypassable via settings or
`--dangerously-skip-permissions`. (Refs: anthropics/claude-code #41526, #34923.)

## ⚠️ Risk

`bypassPermissions` runs everything with zero confirmation, including destructive and
irreversible commands, in **every** directory. Git-tracked dirs are the only real safety
net. The launch-flag equivalent is `claude --dangerously-skip-permissions`.

## Install

```bash
git clone https://github.com/steven068zzy/no-yes.git ~/.claude/skills/no-yes
```
