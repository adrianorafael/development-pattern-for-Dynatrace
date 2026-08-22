# AI Coding Agent Instructions — this repository

This repository **is** the skill *Development Pattern for Dynatrace*. It contains no
application code — only Markdown, templates and one shell script.

## Before editing anything here

Read [`skills/development-pattern-for-dynatrace/SKILL.md`](skills/development-pattern-for-dynatrace/SKILL.md).
The eleven rules it defines apply to this repository too.

## House rules for this repo

- **R1 — no secrets, no tenant data.** Every example uses an obviously-fake, obviously-
  actionable placeholder — `YOUR-ENVIRONMENT`, `YOUR-PLATFORM-TOKEN`,
  `HOST-0000000000000000`. Never a realistic-looking value: a reader must be unable to
  mistake it for something that works. Run the scanner before every commit:

  ```bash
  bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh
  git diff --cached | bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh --stdin
  ```

- **R2 — no invented APIs, especially here.** This skill's credibility rests on its
  inventories being verifiable. Before changing a component name, prop, hook, DQL function
  or `dt-app` command in any reference file, verify it against the published package or
  `developer.dynatrace.com`, and note the version you checked:

  ```bash
  npm view @dynatrace/strato-components version
  grep -oE "export \{[^}]*\}" node_modules/@dynatrace/strato-components/charts/index.d.ts
  ```

  Inventories are **snapshots, not contracts**. Keep the "verified <version>, <date>" line
  next to every one of them, and update it when you re-verify.

- **R10 — no AI co-authorship.** No `Co-Authored-By`, no "Generated with", no session links
  in commits, PRs, README or the page.

- **README ↔ page consistency.** `README.md` and `docs/index.html` state the same things at
  different depths. When one changes, the other changes in the same commit — same
  disclaimer, same rules, same cost numbers.

## Repository layout

```
skills/development-pattern-for-dynatrace/
├── SKILL.md              # the eleven rules, the six-phase pipeline, reference routing
├── references/           # ten documents, loaded on demand
└── assets/
    ├── templates/        # copy-ready files for app repositories
    └── scripts/          # scan-secrets.sh
docs/                     # GitHub Pages site (main branch, /docs folder)
.claude-plugin/           # Claude Code plugin + marketplace manifests
```

## Conventions

- **English** throughout, matching the Dynatrace and Agent Skills ecosystems.
- **Reference documents** open with the rule they enforce and why the failure matters, then
  give commands, then give the checklist. Concrete over abstract.
- **Every absolute claim carries its source** — a `.d.ts` path, a `developer.dynatrace.com`
  URL, or a package version.
- **Templates use `<PLACEHOLDER>` markers** so an unfilled one is visible at a glance.

## Verifying a change

There is no build. Before pushing:

```bash
bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh   # must be clean
bash -n skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh # syntax
# open docs/index.html in a browser and check the layout at 1440px and 375px
```
