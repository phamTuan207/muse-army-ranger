# Worker 3 work log - cli-anything-discord (Phases 2-3)

Wall-clock Asia/Ho_Chi_Minh. T+0 = task receipt 2026-11-24 20:32:39 +07.

- 20:32:39 (T+0m): task received. Read HARNESS.md Phases 2-3 + Principles,
  Phase 1 analysis, ToS research, session-locking guide, repl_skin.py.
- 20:33 (T+1m): verified live: Discord CDP up on 9229 (Chrome/152,
  Electron 44.10.2, discord 1.0.169), test DM page target already open.
  Laptop has Node 26, uv 0.13.9, no system pip, no click, no python
  websocket-client. Decision: transport = Node 26 built-in WebSocket
  (utils/cdp_client.mjs), install via `uv tool install --editable`.
- 20:35 (T+3m): owner follow-up received: minimize Vision/mouse, CDP only.
  Design already CDP-only; added the rule to README.md + SKILL.md.
- 20:35-20:40 (T+3-8m): wrote the package (Click CLI + REPL default,
  session with locked saves, Node CDP client, backend, README, draft
  SKILL.md, TEST.md plan, 5 unit tests). Synced to laptop via rsync.
- 20:40 (T+8m): `uv tool install --editable` (detached) installed
  cli-anything-discord 1.0.0 + click 8.5.1 + prompt-toolkit 3.0.53.
- 20:41 (T+9m): smoke test live. `status` PASS. `servers list` FAIL:
  "server rail not found". Probe found the cause: Vietnamese locale,
  rail aria-label is "Thanh bên máy chủ", not "Servers" (Phase 1's
  English selector). Fixed SERVERS_JS to use the locale-proof
  `data-list-item-id^="guildsnav___"` hook. Retest PASS (13 servers).
- 20:42 (T+10m): `channels list --dm` PASS (8 DMs, test DM present).
  `channels open 865272264831139840` PASS. `messages read --limit 5`
  PASS (5 msgs, id/author/timestamp/content) but 2 authors empty:
  Discord omits the author header on follow-up messages. Fixed READ_JS
  to inherit the last seen author. Retest PASS (all authors filled).
- 20:43 (T+11m): `messages send` to test DM PASS (returned ok, composer
  label "Nhắn @Violet Joestar"). Read-back probe: new message
  1574781159320199189 with exact text. Slate send = Input.insertText +
  Enter rawKeyDown/keyUp. VERIFIED LIVE.
- 20:44 (T+12m): `messages send-file` PASS. Read-back: new message
  1574781352874872916, attachment confirmed on CDN
  (cdn.discordapp.com/attachments/865272264831139840/.../discord-smoke-test.txt).
  VERIFIED LIVE.
- 20:45 (T+13m): `session attach` PASS (note: --json is a group option,
  must precede the subcommand). REPL piped smoke PASS (banner + status).
  `session show` PASS. 5/5 unit tests PASS (pytest 9.2.0 in tool env).
- 20:46 (T+14m): `session start` smoke test + commit to repo.

## Test traffic sent (all to the Violet Joestar test DM only)

1. Text: "[cli-anything-discord smoke test] hello from the CLI"
   (message 1574781159320199189).
2. File: /tmp/discord-smoke-test.txt with caption
   "[cli-anything-discord smoke test] file upload"
   (message 1574781352874872916, attachment on CDN).

No other channel, no public server, no other DM received anything.

## Known minor issues (for the test worker)

- `channels open` reports `channel_title: ""` in DM views (the `h1`
  selector finds no text there); session `current_channel_name` stays
  null. Cosmetic only.
- Server names from the rail carry locale unread-count prefixes
  (e.g. "Tin nhắn chưa đọc, Optimus[Prime]").
- `messages read` returns the viewport window; scrolling up loads older
  messages, so the newest messages can leave the DOM after a scroll.
  A read-back check should scroll to bottom first (see /tmp/probe-last.mjs).
- No Vision/mouse anywhere in this CLI (owner rule 2026-11-24). All
  operations are CDP. Nothing in the CLI required a Vision/mouse step.
