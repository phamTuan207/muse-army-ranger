# TEST.md - cli-anything-discord (HARNESS Phase 4 plan; Phase 6 results appended by test worker)

## Test Inventory Plan (Phase 4, written before test code)

- `test_core.py`: 5 unit tests planned (session save/load round-trip, missing
  file defaults, corrupt file defaults, set_current_channel, repeated-save
  no-corruption).
- `test_full_e2e.py`: E2E tests planned (Phase 5, test worker owns):
  - `status` against the live Discord client (CDP up, target resolved).
  - `servers list`, `channels list` read-only checks.
  - `messages read` on the Violet Joestar test DM (channel 865272264831139840),
    verify JSON shape (id/author/timestamp/content) and count > 0.
  - `messages send` to the test DM only, verify the message appears on a
    follow-up read.
  - `messages send-file` to the test DM only, verify attachment appears.
  - CLI subprocess tests via `_resolve_cli("cli-anything-discord")`,
    `--help` and `--json` on every command.
  - No graceful degradation: tests fail (not skip) if Discord/CDP is down.

## Unit Test Plan

- Module `core/session.py`: `_locked_save_json` (exclusive lock, truncate
  inside lock), `load_session` defaults + corrupt-file tolerance,
  `set_current_channel`, repeated saves.
- Edge cases: missing file, corrupt JSON, concurrent-ish repeated writes.

## E2E Test Plan

- Real workflows: open test DM -> read -> send -> read-back verify ->
  send-file -> read-back verify. All traffic ONLY in the Violet Joestar DM.
- Verify output properties: message JSON fields present, send appears in a
  subsequent read, file message carries an attachment marker/content.
- Print artifact evidence (channel id, message ids, timestamps) for manual
  inspection.

## Test Results (Phase 6 - appended by test worker)

TODO (test worker).
