# Changelog

All notable changes to **Development Pattern for Dynatrace** are documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [Semantic Versioning](https://semver.org/) — applied to this skill the same way
[R12](skills/development-pattern-for-dynatrace/references/release-and-docs-sync.md) requires
of the apps it helps you build. "Breaking" means breaking for the person *using* the skill:
a rule renumbered, a reference removed, a template's contract changed.

The version here matches `.claude-plugin/plugin.json` → `version`.

## [Unreleased]

## [1.1.0] - 2026-08-22

### Added
- **R12 — the version and the docs move with the app.** New reference
  `release-and-docs-sync.md`: semantic versioning stated in app terms, the three rules that
  keep a version trustworthy, a CHANGELOG standard, and a matrix mapping each kind of change
  to the README section, page section, bump level and changelog entry it requires.
- `CHANGELOG.template.md`, with the scope and cost callouts this pattern requires.
- Versioning section in `README.template.md`; "ships in version" and "docs impact" blocks in
  `spec.template.md`; a version/docs section in the review checklist.
- **Dynatrace Docs URL map** — `docs.dynatrace.com` entries for AppEngine, DPS licensing,
  the MCP server and the credential vault.
- **Design foundations and patterns** added to the documentation map:
  `foundations/layout`, `foundations/navigation`, `patterns/loading-saving`,
  `patterns/app-structure`, each with a note on when it is required reading.
- A `gh-pages` publishing workflow, and the two GitHub Pages traps it works around, recorded
  in `readme-and-page.md`.

### Changed
- **The official Dynatrace skills are referenced, never vendored.** `dynatrace-for-ai` is
  updated between sessions, so a local copy looks authoritative while being wrong. The
  instruction is now fetch-on-demand from `raw.githubusercontent.com`, with the stable path
  pattern and a pointer to the repository README as the live catalogue. `.gitignore` blocks a
  vendored copy as a safety net rather than expecting one.
- Phase 0 now asks for the app's **starting version** alongside its identity.
- Phase 6 bumps the version, writes the changelog and updates the README and page **before**
  the approval gate — all in the same commit as the code.
- `scan-secrets.sh` scans per file rather than per line (a full-repository scan went from
  over two minutes to about one second) and checks each matched substring against the
  placeholder allowlist, so one placeholder on a line can no longer mask a real secret
  beside it.

### Fixed
- Reference count was documented as ten when it was already eleven; it is now twelve and
  consistent across `SKILL.md`, the README, the page and `AGENTS.md`.

## [1.0.0] - 2026-08-22

### Added
- First release: `SKILL.md` with eleven rules and a six-phase pipeline, eleven reference
  documents, nine templates, and `scan-secrets.sh`.
- Strato inventories verified against `@dynatrace/strato-components@3.11.3`,
  `strato-icons@2.4.0`, `@dynatrace-sdk/react-hooks@1.14.2` and `dt-app@1.15.0` — including
  the correction that `strato-components-preview` is deprecated in Strato 3.x.
- Project README and GitHub Pages site.

[Unreleased]: https://github.com/adrianorafael/development-pattern-for-Dynatrace/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/adrianorafael/development-pattern-for-Dynatrace/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/adrianorafael/development-pattern-for-Dynatrace/releases/tag/v1.0.0
