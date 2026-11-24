# cli-anything-discord

Message read/send control for the Discord desktop client, built with the
[CLI-Anything](https://github.com/HKUDS/CLI-Anything) 7-phase pipeline
(Apache-2.0). Attribution: the pipeline, HARNESS.md, `repl_skin.py` and the
session-locking pattern come from HKUDS/CLI-Anything.

## What it does

Drives the REAL Discord desktop client (Electron 1.0.169) over Chrome
DevTools Protocol on `127.0.0.1:9229`. No reimplementation, no user token,
no self-bot REST calls. The CLI navigates the client, scrapes the message
DOM, and types into the real composer.

Commands (every command supports `--json`):

```
cli-anything-discord status
cli-anything-discord servers list
cli-anything-discord channels list [--dm | --guild]
cli-anything-discord channels open <channelId>
cli-anything-discord messages read [channelId] [--limit N]
cli-anything-discord messages send [channelId] --text "..."
cli-anything-discord messages send-file [channelId] --file <path> [--text "..."]
cli-anything-discord session start | attach | show | stop --force
cli-anything-discord            # no args: interactive REPL
```

A bare channel id is treated as a DM channel id. `channels open` sets the
current channel in the session file (`~/.cli-anything-discord/session.json`),
so later `messages read/send` can omit the id.

## Hard dependencies

- The official Discord desktop client, logged in, on the same machine.
- Node.js >= 22 (global `WebSocket`, no npm deps) for the CDP transport
  (`utils/cdp_client.mjs`).
- Python >= 3.10, `click`, `prompt-toolkit`.

## Install

```bash
# on the laptop (uv is the available installer; no system pip)
uv tool install --editable /path/to/discord-cli
# or: uv venv ~/.cli-anything-discord-venv && uv pip install --python <venv> -e /path/to/discord-cli
cli-anything-discord status
```

If Discord is not running, `session start` launches it detached with
`--remote-debugging-port=9229` (needs `DISPLAY=:0` XWayland plus the usual
GUI env; the backend handles this).

## ToS risk note (read before use)

This section summarizes research done 2026-11-24 (see
`discord-tos-research.md` in the parent project dir; sources re-checked
there). It is not legal advice. Discord can change its terms at any time.

- Discord's Community Guidelines rule 14: "Do not use self-bots or
  user-bots. Each account must be associated with a human, not a bot."
- Discord's support article on Automated User Accounts says automating
  normal user accounts outside the OAuth2/bot API "is forbidden, and can
  result in an account termination if found".
- The CDP/GUI-automation route is NOT named anywhere in Discord's terms
  (no clause mentions CDP, DevTools, GUI automation or RPA). The honest
  position: probably treated as self-botting if detected, but Discord has
  not published a position on GUI automation specifically. Do NOT present
  this route as safe or compliant.
- The only sanctioned automation path is a proper bot account with a bot
  token, invited to servers. It cannot read the owner's DMs or act as the
  owner, so it does not cover the same use cases.
- The owner approved building this CLI knowing the risk. That approval
  does not change Discord's terms or bind Discord. Keep volume low and
  human-paced, never send bulk or unsolicited messages, never store a
  user token anywhere, and prefer a secondary account you can afford to
  lose for anything beyond personal test traffic.
- This CLI never asks for, reads, or stores a Discord user token or
  password. It only drives the already-logged-in official client on the
  owner's own machine.

## Design notes

- Page target is resolved at runtime via `/json/list` (never hardcoded).
- No Vision, no mouse: this CLI drives Discord purely through CDP
  (HTTP `/json/*` + WebSocket `Runtime.evaluate` / `Input.*` / `DOM.*`).
  Per the owner's standing rule (2026-11-24), Vision/mouse automation is
  used only when no CLI/API/CDP path exists. No step in this CLI uses
  Vision or mouse input, and no such step was added: navigation, reading,
  sending and file upload are all CDP operations. If a future Discord
  change breaks a CDP path and only a Vision/mouse step can replace it,
  that step must be documented here (where, why CDP cannot do it) before
  it is added.
- Selectors prefer stable hooks (roles, aria-labels, `id^="chat-messages-"`);
  hashed class suffixes are fallbacks only and will drift per Discord build.
- Only the viewport window of messages is in the DOM; `messages read`
  scrolls up (bounded, 5 scrolls) to load older messages.
- Sending is irreversible; there is no undo. `session stop` requires
  `--force` and never runs implicitly.
- Presence stays out of this CLI; the existing `discord-window-presence`
  service keeps owning Rich Presence.

## Tests

See `tests/TEST.md` (plan + results). Phase 5 test worker owns the full
suite; `tests/test_core.py` ships unit tests for session handling.
