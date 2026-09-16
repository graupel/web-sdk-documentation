# Changelog

Client-facing changes to the QuadraScan Web SDK. Format follows [Keep a Changelog](https://keepachangelog.com/). Versioning is [semver](https://semver.org/): `MAJOR.MINOR.PATCH`.

- **MAJOR** — breaking: renamed/removed `init()` options, callbacks, payload keys, or defaults that existing integrations depend on
- **MINOR** — additive: new optional flags/fields, or a new step in the iframe
- **PATCH** — bugfixes, copy, performance

## [Unreleased]

## [1.1.0] — 2026-09-15

### Added

- `QuadraScan.version` — string from `package.json`, no `init()` required
- This changelog, published next to Getting Started
- `telemetry` init option — anonymous product analytics inside the scan iframe (default: `true`). Pass `false` to skip Mixpanel entirely: no library load, no Mixpanel cookies or storage, no events. On by default.

### Changed

- `athlete.heightIn` is snapped to the nearest 0.5 inch (e.g. `7.3` → `7.5`). Non-numeric strings such as `"7in"` are rejected.

### Fixed

- `units: 'metric'` now also converts height and weight on the post-scan avatar review (scan summary) card, not only the measurements screen
- Display units are read from session config, so a mid-session storage wipe no longer flips metric back to imperial
- `close()` then `startScan()` no longer drops the new callbacks

## [1.0.0] — 2026-09-02

Baseline. The contract in Getting Started.

### Added

- `QuadraScan.init()`, `startScan()`, and `close()`
- `onComplete` payload: `scanId`, `athlete`, `measurements`
