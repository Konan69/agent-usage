# agent-usage

How much Claude and Codex quota you have left, across every account you pay for.

`claude-usage` asks each provider's usage endpoint with the token already on disk and prints one line per window: about 3 seconds for four accounts, no tokens printed. I run it before deciding which account takes the next big job. `cliproxy-claude-sync` copies a Claude Code sign-in into a CLIProxyAPI auth file without the two fighting over token refresh.

![The terminal runs ./claude-usage --config config.json and prints the used share of the five-hour and seven-day windows for two Claude accounts and the usage window for two Codex accounts](docs/media/hero.gif)

## Install

Python 3, standard library only. Signed-in Claude Code or Codex CLI accounts.

```bash
install -m 755 claude-usage cliproxy-claude-sync ~/.local/bin/
mkdir -p ~/.config/agent-usage
install -m 600 config.example.json ~/.config/agent-usage/config.json   # then point each account at its credential file
```

## Use

```console
$ claude-usage
claude primary seven_day: 74.0% resets 2026-11-04T14:00:00.070937+00:00
claude proxy-1 five_hour: 6.0% resets 2026-11-03T21:39:59.730486+00:00
claude proxy-1 seven_day: 85.0% resets 2026-11-04T21:59:59.730510+00:00
codex coding-1 (pro) 7d: 8% reset_after 158h
codex coding-2 (free) 30d: 52% reset_after 350h
```

A failing account prints `unreadable (<reason>)` and the rest still print.

```bash
cliproxy-claude-sync      # copies the current Claude Code token into the proxy file; silent when already current
```

Run the sync from a timer more often than every 45 minutes: the proxy copy has no refresh token.

## Configuration

`~/.config/agent-usage/config.json`, or `--config <path>`:

```json
{
  "accounts": [
    { "name": "primary", "kind": "claude", "credential_path": "~/.claude/.credentials.json" },
    { "name": "coding-1", "kind": "codex", "credential_path": "~/.codex/auth.json" }
  ],
  "sync": {
    "credential_path": "~/.claude/.credentials.json",
    "target_path": "~/.local/share/cli-proxy-api/auth/claude-example.json"
  }
}
```

`claude-usage` needs only `accounts`; `cliproxy-claude-sync` needs only `sync`, and its `target_path` must already exist.

## Limits

- Both usage endpoints are undocumented and can change without notice.
- No token refresh: an expired token shows `unreadable` until its tool renews it.
- Codex shows only the primary window.
- When the token is near expiry, the sync renews it with one `claude -p` call on `claude-haiku-4-5-20251001`, which counts against that account.

How each script works: [docs/internals.md](docs/internals.md).

## License

MIT
