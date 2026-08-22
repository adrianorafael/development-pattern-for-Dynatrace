# Changelog

All notable changes to **<App Name> for Dynatrace** are documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/), where "breaking" means breaking for
the person running the app — see
`references/release-and-docs-sync.md` in
[Development Pattern for Dynatrace](https://github.com/adrianorafael/development-pattern-for-Dynatrace).

The version here must match `app.config.json` → `app.version`.

<!--
  Write entries for the person running the app, not the person who wrote the diff.
  "Refactored useProblems" tells a user nothing.
  ALWAYS call out, explicitly:
    - scope changes  → everyone must re-consent on next load (MAJOR)
    - cost changes   → refresh interval, default timeframe, projected fields
    - data changes   → persisted state format, required Grail tables
  Sections, in this order, omitting the empty ones:
  Added · Changed · Deprecated · Removed · Fixed · Security
-->

## [Unreleased]

## [1.1.0] - YYYY-MM-DD

### Added
- <New capability, described from the user's side.>

### Changed
- **Auto-refresh now defaults to off.** It ran one Grail query per minute per open tab,
  which was the dominant DPS cost driver. Turn it back on per session if you need it.

### Fixed
- <The symptom the user saw, then the cause.>

## [1.0.0] - YYYY-MM-DD

### Added
- First release.

<!--
  Link references, so version headings become links to the compare view.
-->
[Unreleased]: https://github.com/<user>/<App-Name>-for-Dynatrace/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/<user>/<App-Name>-for-Dynatrace/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/<user>/<App-Name>-for-Dynatrace/releases/tag/v1.0.0
