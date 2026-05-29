---
name: no-yes
description: Force an agentic coding CLI (Claude Code and/or OpenAI Codex) into the most aggressive auto-approve setup — Claude Code global bypassPermissions with all launch dialogs skipped; Codex approval_policy=never + sandbox_mode=danger-full-access. Use when the user wants the agent to never ask for permission again, auto-approve all operations, give maximum/highest permissions, "approve everything automatically", "highest privilege", "skip all confirmations", "bypass permissions", "full access", "yolo mode", "stop asking me anything".
---

# no-yes — force maximum auto-approve (no toggle, always aggressive)

This skill pushes the host CLI to its single most permissive configuration. There is
no "off" — invoking it always pushes to the most aggressive state. To undo, the user
must hand-edit the config or ask explicitly.

It supports two agentic CLIs with the same intent but different config systems:

| | Claude Code | OpenAI Codex |
|---|---|---|
| Config file | `~/.claude/settings.json` (JSON) | `~/.codex/config.toml` (TOML) |
| Keys | `permissions.defaultMode = "bypassPermissions"` (+ skip flags) | `approval_policy = "never"` + `sandbox_mode = "danger-full-access"` |
| Launch flag | `claude --dangerously-skip-permissions` | `codex --yolo` |

## Which targets to configure

1. **Always configure the CLI you are currently running in.**
2. **Also configure any other supported CLI whose config dir already exists** on the
   machine (`~/.claude/` and/or `~/.codex/`). The user's intent is "never ask again,"
   so apply it everywhere it can take effect. Tell the user each target you touched.
   Do **not** create a config dir for a CLI that isn't installed.

---

## Target A — Claude Code (`~/.claude/settings.json`)

Effect (applies to **future sessions** — `defaultMode` is read at startup):
- `permissions.defaultMode = "bypassPermissions"` → auto-yes on every prompt:
  file edits, writes, deletes, arbitrary shell commands, pushes — no confirmation.
- `skipDangerousModePermissionPrompt = true` → skip the one-time "accept dangerous
  mode" dialog on launch.
- `skipAutoPermissionPrompt = true` → skip the auto-mode opt-in dialog.

For the **current** Claude Code session this can't change the live permission mode
(set at startup). Tell the user to press **Shift+Tab** until the footer reads
`bypass permissions`, or just restart.

Steps:
1. **Read** `~/.claude/settings.json`. If it doesn't exist, create it as
   `{ "permissions": { "defaultMode": "bypassPermissions" }, "skipDangerousModePermissionPrompt": true, "skipAutoPermissionPrompt": true }`.
2. **Merge — never overwrite the whole file.** Preserve every existing key (hooks,
   env, plugins, statusLine, theme, the existing `permissions.allow` array, etc.).
   Edit only:
   - `permissions.defaultMode` → `"bypassPermissions"`
   - top-level `skipDangerousModePermissionPrompt` → `true`
   - top-level `skipAutoPermissionPrompt` → `true`
   - Remove any `permissions.deny` / `permissions.ask` entries that would re-introduce
     prompts (the "most aggressive" mandate). Leave `permissions.allow` intact.
3. **Validate** the file is still valid JSON (`python3 -c "import json;json.load(open(...))"`).
   A malformed settings.json silently disables ALL settings — fix before finishing.

## Target B — OpenAI Codex (`~/.codex/config.toml`)

Effect: removes both the approval prompts and the sandbox.
- `approval_policy = "never"` → Codex never pauses to ask for command/edit approval.
- `sandbox_mode = "danger-full-access"` → no filesystem boundary, no network isolation.

Steps:
1. **Read** `~/.codex/config.toml`. If it doesn't exist, create it with just:
   ```toml
   approval_policy = "never"
   sandbox_mode = "danger-full-access"
   ```
2. **Merge — preserve every existing key** (`model`, `model_provider`, `[mcp_servers.*]`
   tables, profiles, etc.). Set/replace only the two keys above.
   - These are **top-level** keys: they must appear **before any `[table]` header**,
     otherwise TOML scopes them inside that table and they're ignored. If they already
     exist under a `[profiles.*]` or other section, also set them at top level.
   - If a `[sandbox_workspace_write]` or other narrower sandbox config exists, it is
     overridden by `danger-full-access` but harmless to leave in place.
3. **Validate** the file still parses as TOML
   (`python3 -c "import tomllib;tomllib.load(open('...','rb'))"`).

Codex reads `config.toml` at startup, so this also takes effect on the **next** Codex
launch. The one-shot launch equivalent is `codex --yolo`.

---

## Confirm to the user

State which target(s) you wrote, that maximum auto-approve is now configured and takes
effect on the next launch of each, and (for the current Claude Code session) to use
**Shift+Tab** / restart.

## Known exception — Claude Code protected directories still prompt

Even with `bypassPermissions` active, a **hardcoded guardrail** (since v2.1.78) still
prompts before writing to protected paths — `.git/`, `.claude/`, `.vscode/`, `.husky/` —
and before Claude edits its own config files (e.g. `SKILL.md`). Neither
`defaultMode: bypassPermissions` nor `--dangerously-skip-permissions` bypasses this;
it surfaces as a separate per-session "allow Claude to modify its config files" dialog.
So "auto-yes on **everything**" is true for ordinary edits/writes/shell, but **not** for
these protected dirs — tell the user they will still get one prompt there, and it is
expected, not a sign the skill failed. (Refs: anthropics/claude-code issues #41526, #34923.)
Codex has no equivalent protected-path carve-out — `danger-full-access` is exactly that.

## Reality check (state this once, do not block)

This runs everything with zero confirmation, including destructive and irreversible
commands, in **every** directory — not just sandboxes. Codex's `danger-full-access`
goes further than Claude's bypass: it also removes the **network isolation** and
filesystem sandbox, so the agent can reach anything your user account can reach. The
user has chosen this explicitly; apply it. Git-tracked dirs are the only real safety net.
