# Changelog

All notable changes to titan-framework are documented here.

---

## [Unreleased]

_No unreleased changes._

## [1.0.0b2] - 2026-09-04

### Bug Fixes

- Inline buttons now require exactly one supported action:
  `callback_data` or `url`. Creating a button with neither or both now raises
  `TitanError` before a payload is built or a Telegram request is sent.
- Async `on_offset` callbacks are rejected with `TitanError` before polling
  starts. The lint rule remains as defensive validation for invalid internal
  state.
- The error-handler registration contract is now explicit and tested:
  each bot has one slot, and the last handler replaces the previous one
  without a warning or exception.
- Unhandled errors and errors raised by an error handler are logged through the
  `titan` logger with their tracebacks instead of being printed to stdout.
- **B2-008 — `bot.include()` preflight:** `bot.include()` now validates all
  router registrations and checks command and callback conflicts before
  mutating bot state. Validation or conflict failures no longer leave partial
  state from that include attempt. This is preflight protection before
  mutation, not a guarantee of full atomicity.

### Improvements

- Polling now has explicit per-run lifecycle ownership for chat workers and
  direct update-handler tasks.
- Polling shutdown now drains accepted chat updates, gives accepted handler
  tasks a bounded opportunity to finish, cancels tasks that remain pending,
  observes task results and exceptions, and closes the Telegram API session
  only after lifecycle cleanup.
- Cancelling `run_async()` preserves the caller-visible
  `asyncio.CancelledError` after lifecycle cleanup. Handler cancellation during
  shutdown is treated as lifecycle control rather than user-facing failure.

### Documentation / Packaging

- Added the lifecycle ownership, shutdown, cancellation, and task-failure
  contract to the project documentation.
- Aligned keyboard, `on_offset`, error-handler, error-logging, and
  `bot.include()` documentation with the implemented behavior.
- Declared the custom WASL license in package metadata and added its SPDX
  identifier to `LICENSE`.

### Tests / Internal Changes

- Centralized polling task ownership in the internal `LifecycleRegistry`.
- Expanded coverage for lifecycle gates, direct polling tasks, bounded
  shutdown, task-failure observation, keyboard validation, `on_offset`
  validation, error-handler replacement, and `bot.include()` preflight.
- Cold-import inspector coverage now restores the original `titan` modules
  after the test.

## [1.0.0b1] - 2026-08-11

### Packaging

- The distribution package is named `titan-framework`.
- The Python import path remains `titan`, so existing
  `from titan import ...` code is unchanged.
- The source distribution contains only the package source and the files
  required to build and identify the release.
- Tests, documentation, examples, ecosystem material, and development-only
  project files are excluded from release artifacts.

### Fixed

- `AliasMap.register()` now fails fast with `TitanError` when an alias
  conflicts with an existing `Context` attribute.
- Middleware return values now produce a runtime warning because they are
  ignored by the framework contract.
- Startup `get_me()` failures now produce a warning instead of being silently
  swallowed.
- Documentation and examples use `AliasMap` and `fetch_permissions()`.

### Added

- **Titan Atlas** (`titan.atlas`) — a deterministic architectural knowledge
  layer with `search()`, `explain()`, `rules()`, and `decisions()`.
- **Performance Profiler** (`titan.profiler`) for measuring update handling
  time in a controlled environment.
- **Message Links Protocol** for stable, addressable identities for sent
  messages.
- **Runtime Contract Validator** for validating handler registrations when
  they are declared.

### Runtime Semantics

- Per-chat update dispatch advances the polling offset when an update is
  accepted for processing, so a handler waiting for an `ask()` response does
  not block polling.

## [1.0.0a2] - 2026-07-18

Alpha patch release. No new features or architectural changes.

### Fixed

- Corrected the aliases documentation in the Arabic and English READMEs.
- Corrected the permissions API documentation and moderation example.

## [1.0.0a1] - 2026-07-01

First public alpha release. The public API is stable and contract-frozen.

### Public API

- `Titan` — core bot class.
- `Router` — code organization tool for splitting handlers across files.
- `InlineKeyboard` / `InlineButton` — inline keyboard builders.
- `TitanError` / `TelegramError` — error types.

### Bot Methods

- `bot.on(event)` — raw event handler.
- `bot.command(name)` — command handler.
- `bot.callback(data)` — inline button handler.
- `bot.middleware` — pre-handler middleware.
- `bot.include(router)` — merge a router into the bot.
- `bot.run()` / `bot.run_async()` — entry points.
- `bot.telegram` — direct Telegram API adapter.

### Known Sharp Edges

- An empty `callback_data` registers successfully but is never matched.
- The README documents additional safeguards that are intentionally not
  enforced automatically.