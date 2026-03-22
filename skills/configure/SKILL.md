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
All account data is stored under `~/.claude/channels/wechat/<account-name>/`.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### `<account-name>` — login a named account (REQUIRED)

1. **Check status** — read `~/.claude/channels/wechat/<account-name>/account.json`.
   - If exists: show botId (masked), userId, savedAt. Ask if they want to re-login.
   - If missing: proceed to login.

2. **Login** — run login with account name:
   ```
   bun ${CLAUDE_PLUGIN_ROOT}/test-login.ts --account <account-name>
   ```

3. **After success** — write `.wechat-account` to the current project dir:
   ```
   echo "<account-name>" > .wechat-account
   ```
   Tell the user: *"WeChat account '<account-name>' connected! Restart Claude Code to activate."*

### No args — prompt the user

Tell the user they must specify an account name:
*"请指定账号名，例如: /wechat:configure work"*

Then list existing accounts (see `list` below).

### `list` — list all accounts

Run: `ls ~/.claude/channels/wechat/`
For each subdirectory that contains `account.json`, show:
- Account name (directory name)
- botId (first 12 chars + `...`)
- Login time

### `status` — check current account

1. Read `.wechat-account` from the current project dir to get account name.
2. If not found, tell user no account bound to this directory.
3. If found, read `~/.claude/channels/wechat/<account-name>/account.json` and show status.

### `logout <account-name>` — remove credentials

1. Delete `~/.claude/channels/wechat/<account-name>/account.json`
2. Delete `~/.claude/channels/wechat/<account-name>/sync-buf.txt`
3. Confirm: *"Logged out. Run /wechat:configure <account-name> to re-login."*

### `reset <account-name>` — full reset

1. Delete entire `~/.claude/channels/wechat/<account-name>/` directory.
2. Confirm: *"Reset complete."*

---

## Multi-account usage

Account is auto-detected from `.wechat-account` file in project dir (via `CLAUDE_PROJECT_ROOT`).

```bash
# Different projects, different WeChat accounts — zero config after initial setup
cd ~/project-a && claude --dangerously-load-development-channels plugin:wechat@claude-channel-wechat
cd ~/project-b && claude --dangerously-load-development-channels plugin:wechat@claude-channel-wechat
```

## Implementation notes

- The login script (`test-login.ts`) MUST run with `bun`.
- `--account <name>` stores credentials in `~/.claude/channels/wechat/<name>/`.
- There is NO root-level account.json. Every account lives in a named subdirectory.
- The MCP server reads `.wechat-account` from `CLAUDE_PROJECT_ROOT`. If not found, it waits.
