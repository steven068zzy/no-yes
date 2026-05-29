---
name: no-yes
description: Force Claude Code into the most aggressive auto-approve setup — global bypassPermissions (auto-yes on everything, highest privilege) with all confirmation dialogs skipped. Use when the user wants Claude Code to never ask for permission again, auto-approve all operations, give maximum/highest permissions, "approve everything automatically", "highest privilege", "skip all confirmations", "bypass permissions", "stop asking me anything".
---

# no-yes — force maximum auto-approve (no toggle, always aggressive)

This skill writes the single most permissive configuration into
`~/.claude/settings.json`. There is no "off" — invoking it always pushes to the
most aggressive state. To undo, the user must hand-edit settings or ask explicitly.

Effect (applies to **future sessions** — `defaultMode` is read at startup):
- `permissions.defaultMode = "bypassPermissions"` → auto-yes on every prompt:
  file edits, writes, deletes, arbitrary shell commands, pushes — no confirmation.
- `skipDangerousModePermissionPrompt = true` → skip the one-time "accept dangerous
  mode" dialog on launch.
- `skipAutoPermissionPrompt = true` → skip the auto-mode opt-in dialog.

For the **current** session, this can't change the live permission mode (set at
startup). Tell the user to press **Shift+Tab** until the footer reads
`bypass permissions`, or just restart.

## Steps

1. **Read** `~/.claude/settings.json` (Read tool). If it doesn't exist, create it as
   `{ "permissions": { "defaultMode": "bypassPermissions" }, "skipDangerousModePermissionPrompt": true, "skipAutoPermissionPrompt": true }`.

2. **Merge — never overwrite the whole file.** Preserve every existing key (hooks,
   env, plugins, statusLine, theme, the existing `permissions.allow` array, etc.).
   Edit only:
   - `permissions.defaultMode` → `"bypassPermissions"`
   - top-level `skipDangerousModePermissionPrompt` → `true`
   - top-level `skipAutoPermissionPrompt` → `true`
   - Remove any `permissions.deny` / `permissions.ask` entries that would re-introduce
     prompts (this is the "most aggressive" mandate). Leave `permissions.allow` intact.

3. **Validate** the file is still valid JSON (e.g. `python3 -c "import json;json.load(open(...))"`).
   A malformed settings.json silently disables ALL settings — fix before finishing.

4. **Confirm** to the user: state that maximum auto-approve is now written and takes
   effect on the next launch; for the current session use **Shift+Tab** / restart.

## Known exception — protected directories still prompt

Even with `bypassPermissions` active, a **hardcoded guardrail** (since v2.1.78) still
prompts before writing to protected paths — `.git/`, `.claude/`, `.vscode/`, `.husky/` —
and before Claude edits its own config files (e.g. `SKILL.md`). Neither
`defaultMode: bypassPermissions` nor `--dangerously-skip-permissions` bypasses this;
it surfaces as a separate per-session "allow Claude to modify its config files" dialog.
So "auto-yes on **everything**" is true for ordinary edits/writes/shell, but **not** for
these protected dirs — tell the user they will still get one prompt there, and it is
expected, not a sign the skill failed. (Refs: anthropics/claude-code issues #41526, #34923.)

## Reality check (state this once, do not block)

`bypassPermissions` runs everything with zero confirmation, including destructive and
irreversible commands. This config is global, so it applies in every directory — not
just sandboxes. The user has chosen this explicitly; apply it. Mention `git`-tracked
dirs are the only real safety net and that the launch-flag equivalent is
`claude --dangerously-skip-permissions`.
