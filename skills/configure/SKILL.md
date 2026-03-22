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
Credentials are stored in `~/.claude/channels/wechat/<path-hash>/accounts.json`, keyed by project directory.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args (default) — login

1. **Run login**:
   ```
   bun ${CLAUDE_PLUGIN_ROOT}/test-login.ts --project-root ${CLAUDE_PROJECT_ROOT}
   ```

2. **After success** — tell the user: *"WeChat connected! Restart Claude Code to activate."*

### `list` — list all accounts for this project

1. Compute path hash from `${CLAUDE_PROJECT_ROOT}`.
2. Read `~/.claude/channels/wechat/<path-hash>/accounts.json`.
3. For each account, show:
   - User ID (key)
   - botId (first 12 chars + `...`)
   - Login time
   - Whether it's the default

### `status` — check current account

1. Read `~/.claude/channels/wechat/<path-hash>/accounts.json`.
2. Show the default account's info (botId, userId, savedAt).
3. If no accounts.json found, tell user to run `/wechat:configure`.

### `switch <user_id>` — change default account

1. Read `~/.claude/channels/wechat/<path-hash>/accounts.json`.
2. Set `default` to `<user_id>`.
3. Write back. Confirm: *"Default switched. Restart Claude Code to activate."*

### `logout [user_id]` — remove credentials

1. If `user_id` given, remove that entry from `accounts.json`.
2. If no `user_id`, remove the default account.
3. If the removed account was the default, clear the `default` field.
4. Confirm: *"Logged out. Run /wechat:configure to re-login."*

### `reset` — full reset for this project

1. Delete entire `~/.claude/channels/wechat/<path-hash>/` directory.
2. Confirm: *"Reset complete."*

---

## How path-hash works

```typescript
import { createHash } from 'crypto'
function pathHash(dir: string): string {
  return createHash('sha256').update(dir).digest('hex').slice(0, 12)
}
// pathHash('/home/fish/project-a') → 'a1b2c3d4e5f6'
```

The project root (`CLAUDE_PROJECT_ROOT`) is hashed to a 12-char hex string. This means:
- Different projects get separate credential stores
- No files pollute the project directory
- Moving a project directory invalidates the hash (re-scan needed)

## Implementation notes

- The login script (`test-login.ts`) MUST run with `bun`.
- `--project-root` tells the script which directory to hash.
- `WECHAT_USER_ID` env var overrides the default account selection at runtime.
