---
name: "cli-anything-discord"
description: "Read and send Discord messages by driving the real Discord desktop client over CDP. Status, servers, channels, messages, session. Every command supports --json."
---

# cli-anything-discord (DRAFT SKILL - Phase 6.5 test worker to finish)

> DRAFT: written in Phase 3 as a skeleton. The Phase 6.5 worker must verify
> every command against the live CLI, fill in real outputs, and move the
> canonical copy to `skills/cli-anything-discord/SKILL.md`.

## Install

- Hard dependencies: official Discord desktop client (logged in, same
  machine), Node.js >= 22, Python >= 3.10 with `click` and `prompt-toolkit`.
- `uv tool install --editable <path-to-discord-cli>` (laptop has no system pip).
- `cli-anything-discord session start` launches Discord with CDP on port 9229
  if it is not already running. Never run `session stop` without `--force`.

## ToS risk (must tell the user)

Automating a normal Discord user account is forbidden by Discord's
Community Guidelines (rule 14, self-bots) and Discord's support article
says it "can result in an account termination if found". The CDP route
(driving the official client) is not named in the terms, but is probably
treated as self-botting if detected. Do not present it as safe. The only
sanctioned automation is a bot token, which cannot read owner DMs. This CLI
never uses, asks for, or stores a user token.

## Commands

All commands accept a global `--json` flag for machine-readable output.
A bare channel id is a DM channel id. `channels open <id>` sets the
current channel; later `messages` commands can omit the id.

```
cli-anything-discord status
cli-anything-discord servers list
cli-anything-discord channels list [--dm | --guild]
cli-anything-discord channels open <channelId>
cli-anything-discord messages read [channelId] [--limit N]
cli-anything-discord messages send [channelId] --text "..."
cli-anything-discord messages send-file [channelId] --file <path> [--text "..."]
cli-anything-discord session start | attach | show | stop --force
cli-anything-discord            # REPL (default with no args)
```

## Usage notes for agents

- Always run `status` first to confirm Discord + CDP are up before reading
  or sending.
- Test traffic goes ONLY to the owner-designated test DM unless the owner
  explicitly names another channel. Never send to a public server or
  another DM unasked.
- `messages read` returns only the viewport window of messages (scroll
  loads a bounded amount of older history). `count` vs `total_visible`
  shows whether the result was truncated by `--limit`.
- Sending is irreversible; there is no undo and no delete command.
- If a command fails with "composer not found" or "No messages found",
  the Discord DOM probably changed; report the exact error, do not retry
  blindly.
- No Vision, no mouse: every operation is CDP. Never add a Vision/mouse
  step unless CDP provably cannot do it, and document it in README.md
  first (where, why no CDP alternative).
- Pipeline attribution: built with HKUDS/CLI-Anything (Apache-2.0).
