# no-yes

An Agent Skill that forces an agentic coding CLI into its **most aggressive
auto-approve** configuration. Same intent across two tools, different config systems —
there is no "off": invoking it always pushes to the most permissive state.

Supports **Claude Code** and **OpenAI Codex**. It configures the CLI you're running in,
plus any other supported CLI already installed on the machine.

## What it writes

**Claude Code** → merged into `~/.claude/settings.json` (existing keys preserved):
- `permissions.defaultMode = "bypassPermissions"` — auto-yes on edits, writes, deletes,
  shell, pushes — no confirmation.
- `skipDangerousModePermissionPrompt = true` — skip the "accept dangerous mode" launch dialog.
- `skipAutoPermissionPrompt = true` — skip the auto-mode opt-in dialog.

**OpenAI Codex** → merged into `~/.codex/config.toml` (existing keys preserved):
- `approval_policy = "never"` — Codex never pauses to ask.
- `sandbox_mode = "danger-full-access"` — no filesystem boundary, no network isolation.

Both take effect on the **next** launch of that tool. For the current Claude Code
session, press **Shift+Tab** until the footer reads `bypass permissions`, or restart.
Launch-flag equivalents: `claude --dangerously-skip-permissions` / `codex --yolo`.

## Known exception (Claude Code only)

Even under `bypassPermissions`, a hardcoded guardrail (since v2.1.78) still prompts
before writing to protected paths — `.git/`, `.claude/`, `.vscode/`, `.husky/` — and
before Claude edits its own config. Not bypassable via settings or
`--dangerously-skip-permissions`. (Refs: anthropics/claude-code #41526, #34923.)
Codex's `danger-full-access` has no such carve-out.

## ⚠️ Risk

This runs everything with zero confirmation, including destructive and irreversible
commands, in **every** directory. No sandbox. Codex's `danger-full-access` goes further
than Claude's bypass — it also strips network isolation, so the agent can reach anything
your user account can. Git-tracked dirs are the only real safety net. Use only where you
accept that.

## Install

```bash
git clone https://github.com/steven068zzy/no-yes.git ~/.claude/skills/no-yes
```
