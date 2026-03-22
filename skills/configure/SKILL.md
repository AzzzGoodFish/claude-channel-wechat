---
name: configure
description: Set up the WeChat channel — scan QR code to login. Supports multiple accounts. Use when the user wants to connect WeChat, asks "how do I set this up", or needs to re-login.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(bun *)
  - Bash(ls *)
  - Bash(mkdir *)
  - Bash(rm *)
  - Bash(cat *)
---

# /wechat:configure — WeChat Channel Setup

Runs QR code login for the WeChat channel.
Credentials are stored in `~/.claude/channels/wechat/accounts.json`.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args (default) — login

1. **Run login**:
   ```
   bun ${CLAUDE_PLUGIN_ROOT}/test-login.ts
   ```

2. **After success** — tell the user: *"WeChat connected! Restart Claude Code to activate."*

### `list` — list all accounts

1. Read `~/.claude/channels/wechat/accounts.json`.
2. For each account, show:
   - User ID (key)
   - botId (first 12 chars + `...`)
   - Login time
   - Whether it's the default

### `status` — check current account

1. Read `~/.claude/channels/wechat/accounts.json`.
2. Show the default account's info (botId, userId, savedAt).
3. If no accounts.json found, tell user to run `/wechat:configure`.

### `switch <user_id>` — change default account

1. Read `~/.claude/channels/wechat/accounts.json`.
2. Set `default` to `<user_id>`.
3. Write back. Confirm: *"Default switched. Restart Claude Code to activate."*

### `logout [user_id]` — remove credentials

1. If `user_id` given, remove that entry from `accounts.json`.
2. If no `user_id`, remove the default account.
3. If the removed account was the default, clear the `default` field.
4. Confirm: *"Logged out. Run /wechat:configure to re-login."*

### `reset` — full reset

1. Delete `~/.claude/channels/wechat/accounts.json` and `sync-buf.txt`.
2. Confirm: *"Reset complete."*

---

## Implementation notes

- The login script (`test-login.ts`) MUST run with `bun`.
- `WECHAT_USER_ID` env var overrides the default account selection at runtime.
- `WECHAT_STATE_DIR` env var overrides the state directory (default: `~/.claude/channels/wechat`).
