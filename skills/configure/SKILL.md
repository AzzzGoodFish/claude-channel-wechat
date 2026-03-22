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

# /wechat:configure — WeChat Channel Setup (Multi-Account)

Runs QR code login for the WeChat channel. Supports multiple accounts.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status of current account and login

1. **Check status** — read `~/.claude/channels/wechat/account.json` (default account).
   - If exists: show botId (masked), userId, savedAt. Ask if they want to
     re-login.
   - If missing: proceed to login.

2. **Login** — run the login script:
   ```
   bun ${CLAUDE_PLUGIN_ROOT}/test-login.ts
   ```

3. **After success** — tell the user:
   *"WeChat connected! Send a message from WeChat to test."*

### `<account-name>` — login a specific account

1. **Check status** — read `~/.claude/channels/wechat/<account-name>/account.json`.
   - If exists: show status. Ask if they want to re-login.
   - If missing: proceed to login.

2. **Login** — run login with account name:
   ```
   bun ${CLAUDE_PLUGIN_ROOT}/test-login.ts --account <account-name>
   ```

3. **After success** — write the account name to `.wechat-account` in the current working directory:
   ```
   echo "<account-name>" > .wechat-account
   ```
   Then tell the user:
   *"WeChat account '<account-name>' connected! This directory is now bound to this account. Restart Claude Code to activate."*

### `list` — list all accounts

1. List all directories under `~/.claude/channels/wechat/` that contain `account.json`.
2. Also check `~/.claude/channels/wechat/account.json` for the default account.
3. Show: account name, botId (masked), login time.

### `status` — check current account state

Read and display `~/.claude/channels/wechat/account.json` (or the account specified by `WECHAT_ACCOUNT` env var at `~/.claude/channels/wechat/$WECHAT_ACCOUNT/account.json`). Show:
- Account name (default or from env)
- Login state (configured / not configured)
- botId (first 12 chars + `...`)
- Last login time

### `logout` or `logout <account-name>` — remove credentials

For default: delete `~/.claude/channels/wechat/account.json` and `sync-buf.txt`.
For named: delete `~/.claude/channels/wechat/<account-name>/account.json` and `sync-buf.txt`.
Confirm: *"Logged out. Run /wechat:configure to re-login."*

### `reset` or `reset <account-name>` — full reset

For default: delete `~/.claude/channels/wechat/account.json` and `sync-buf.txt`.
For named: delete entire `~/.claude/channels/wechat/<account-name>/` directory.
Confirm: *"Reset complete. Run /wechat:configure to start fresh."*

---

## Multi-account usage

Each Claude Code instance can use a different WeChat account. Account is resolved in order:

1. `--account <name>` CLI arg (server.ts / test-login.ts)
2. `WECHAT_ACCOUNT` env var
3. `.wechat-account` file in current working directory
4. Falls back to `default`

**Usage**: run `/wechat:configure <name>` — it writes `.wechat-account` to the project dir. The MCP server reads it via `CLAUDE_PROJECT_DIR` automatically. No env vars needed.

```bash
# Just start Claude Code in any project dir — account auto-detected from .wechat-account
cd ~/project-a && claude --dangerously-load-development-channels plugin:wechat@claude-channel-wechat
cd ~/project-b && claude --dangerously-load-development-channels plugin:wechat@claude-channel-wechat
```

Account resolution order:
1. `--account <name>` CLI arg
2. `WECHAT_ACCOUNT` env var
3. `.wechat-account` file in project dir (via `CLAUDE_PROJECT_DIR`)
4. Falls back to `default`

All account data stored under `~/.claude/channels/wechat/<account-name>/`.

## Implementation notes

- The login script (`test-login.ts`) MUST run with `bun`.
- `--account <name>` stores credentials in `~/.claude/channels/wechat/<name>/`.
- Default account (no name) uses `~/.claude/channels/wechat/` directly for backward compatibility.
- The MCP server detects account.json automatically — no restart needed after login.
- `WECHAT_ACCOUNT` env var is read by the MCP server subprocess at startup.
