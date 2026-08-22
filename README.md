# Development Pattern for Dynatrace

🌐 **Project page:** https://adrianorafael.github.io/development-pattern-for-Dynatrace/

**Development Pattern for Dynatrace** is an [Agent Skill](https://agentskills.io) created
and maintained by [@adrianorafael](https://github.com/adrianorafael) on GitHub.

It is a **guardrail** for building native Dynatrace Apps with an AI coding agent. Vibecoding
a Dynatrace App is fast and pleasant right up to the moment the agent invents a component
prop, styles a button with Tailwind, ships a query that returns nothing, or pushes a tenant
URL to a public repository. This skill exists to make those four things structurally
difficult instead of merely discouraged.

It keeps the agent inside [developer.dynatrace.com](https://developer.dynatrace.com/) and the
[Strato Design System](https://developer.dynatrace.com/design/), makes it verify every API
against the installed `.d.ts` files before using it, forces DQL to be executed against a real
tenant before it ships, and sanitizes everything on its way to Git.

> ⚠️ **Disclaimer**
>
> This skill is provided by the developer with **no affiliation with Dynatrace** and **no responsibility** for any failures, issues, or consumption of resources and licenses. It is a **study version with no official support**.
>
> Deciding to download and use it — and to run, deploy or publish the apps it helps you build — is **at the user's own risk**. You are equally free to improve and expand it.

## Overview

The skill is a `SKILL.md` plus twelve reference documents, a set of copy-ready templates,
and a secret scanner. It loads into any agent that supports the open
[Agent Skills](https://agentskills.io) format — Claude Code, Cursor, GitHub Copilot,
OpenCode, Gemini CLI and others — and takes effect the moment the agent starts working on a
Dynatrace App.

It encodes **twelve non-negotiable rules** and a **six-phase pipeline** with three hard
approval gates: no spec without research, no deploy without a named target, no push without
a sanitize pass.

It ships **no copy** of anyone else's documentation. The Dynatrace docs and Dynatrace's own
agent skills are referenced by URL and fetched at the moment they are needed — a vendored
copy goes stale between sessions while still looking authoritative.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Connecting to Dynatrace](#connecting-to-dynatrace)
4. [How to Use It](#how-to-use-it)
5. [The Twelve Rules](#the-twelve-rules)
6. [The Pipeline](#the-pipeline)
7. [What It Prevents](#what-it-prevents)
8. [Important Behaviors](#important-behaviors)
9. [Cost & Consumption (DPS)](#cost--consumption-dps)
10. [Repository Structure](#repository-structure)
11. [Templates & Tools](#templates--tools)
12. [Versioning](#versioning)
13. [Keeping It Current](#keeping-it-current)

## Prerequisites

- An AI coding agent that supports the [Agent Skills](https://agentskills.io) format
  (Claude Code, Cursor, Copilot, OpenCode, Gemini CLI, …)
- Node.js >= 18 and npm >= 9 — for the Dynatrace App Toolkit (`dt-app`)
- A Dynatrace environment with Grail, for the apps you build
- *Recommended:* the [Dynatrace MCP server](https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server)
  or [dtctl](https://github.com/dynatrace-oss/dtctl), so DQL can be validated against a live
  tenant (rule R6)
- Network access to `developer.dynatrace.com`, `docs.dynatrace.com` and GitHub — the skill
  reads them live rather than quoting a cached copy

## Installation

### Skills package (recommended)

```bash
npx skills add adrianorafael/development-pattern-for-Dynatrace
```

### Claude Code plugin

```bash
claude plugin marketplace add adrianorafael/development-pattern-for-Dynatrace
claude plugin install development-pattern-for-dynatrace@development-pattern-for-dynatrace
```

### Manual

Copy the skill directory into your agent's skills path — `.claude/skills/`,
`.agents/skills/`, `.cursor/skills/`, whichever your tool reads:

```bash
git clone https://github.com/adrianorafael/development-pattern-for-Dynatrace.git
cp -r development-pattern-for-Dynatrace/skills/development-pattern-for-dynatrace \
      .claude/skills/development-pattern-for-dynatrace
```

### The official Dynatrace skills — referenced, not bundled

This skill covers *how to build and publish an app*. Dynatrace's own skills at
[Dynatrace/dynatrace-for-ai](https://github.com/Dynatrace/dynatrace-for-ai) cover *what the
data means* — DQL syntax, Grail semantics, entity models, platform costs.

**This repository ships no copy of them, deliberately.** Dynatrace updates those files
between your sessions; a vendored copy looks authoritative while being wrong. The skill's
instruction is to fetch the specific file it needs, fresh, at the moment it needs it:

```bash
curl -sSL https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/skills/dt-dql-essentials/SKILL.md
```

If you prefer them installed in your own agent as well, that is your environment's business
and works fine alongside this skill:

```bash
npx skills add dynatrace/dynatrace-for-ai
```

## Connecting to Dynatrace

The skill provides knowledge and guardrails. To let it **validate** DQL (rule R6), pair it
with live platform access:

```bash
# .env — gitignored. Real values live ONLY here.
DT_ENVIRONMENT=https://YOUR-ENVIRONMENT.apps.dynatrace.com
DT_PLATFORM_TOKEN=dt0s16.YOUR-PLATFORM-TOKEN
```

```jsonc
// .mcp.json — committed, with ${ENV} references, never literal values
{
  "mcpServers": {
    "dynatrace": {
      "type": "http",
      "url": "${DT_ENVIRONMENT}/platform-reserved/mcp-gateway/v0.1/servers/dynatrace-mcp/mcp",
      "headers": { "Authorization": "Bearer ${DT_PLATFORM_TOKEN}" }
    }
  }
}
```

Replace `YOUR-ENVIRONMENT` with your own tenant ID — the URL has the form
`https://<tenant-id>.apps.dynatrace.com`, copied from your Dynatrace address bar. Grant the
platform token the **minimum** scopes for the queries your app runs. See the
[Dynatrace MCP server docs](https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server)
for the required set.

Without MCP the skill still works — it simply labels every unvalidated query
⚠️ *Unvalidated* rather than pretending otherwise.

## How to Use It

Once installed, the skill triggers on its own when you mention a Dynatrace App, `dt-app`,
Strato, AppEngine, DQL or Grail. You can also invoke it explicitly.

```
"Build me a Dynatrace app that shows Kubernetes workload health by namespace."
```

The agent will then, in order:

1. **Ask the five bootstrap questions** — repository, target tenant, MCP availability, app
   identity, publication intent — in one batch.
2. **Research** `developer.dynatrace.com` depth-first, following in-page links, and record
   an evidence log of what it actually read.
3. **Write `specs/<slug>.md`** and stop for your approval — including the exact DQL, the
   chosen visualization *and why that one*, the data contract, and the cost estimate.
4. **Build** with Strato only, verifying every component and prop against `node_modules`.
5. **Validate** — execute the DQL, show you the real records, assert the data shape, run
   `tsc`, ESLint and the security review.
6. **Run, deploy, publish** — each behind an explicit gate, with a sanitize pass before Git.

Add [`AGENTS.template.md`](skills/development-pattern-for-dynatrace/assets/templates/AGENTS.template.md)
to your app repository as `AGENTS.md` so future sessions pick the pattern up automatically.

## The Twelve Rules

| # | Rule |
| --- | --- |
| **R1** | **Never publish a secret.** No tokens, `.env`, tenant IDs or URLs, emails, customer names. Placeholders instead. |
| **R2** | **Never invent an API.** Every component, prop, icon, hook, DQL function and CLI flag is verified against `node_modules/**/*.d.ts` or the docs, in-session. |
| **R3** | **Strato only.** No MUI, Ant, Chakra, Tailwind, Recharts, Chart.js, ECharts or raw D3. |
| **R4** | **Research before spec.** Read the section *and follow its links*. Never answer from memory alone. |
| **R5** | **Spec before code**, with an explicit approval gate. |
| **R6** | **Every DQL is executed** against a live tenant before it ships. |
| **R7** | **Every visualization is fed a verified data shape** — converted, asserted, then rendered. |
| **R8** | **No monotonous screens.** Chart type follows the data's shape and the question, not habit. |
| **R9** | **Review AI-written code like hostile code**, against a security and quality checklist. |
| **R10** | **No AI co-authorship** in commits, PRs, README, page or comments — unless you ask for it. |
| **R11** | **Naming convention `<App Name> for Dynatrace`** for repository, README, page and Hub listing. |
| **R12** | **The version and the docs move with the app.** Every publication bumps `app.version` per SemVer; every push that changes behaviour, setup or cost updates the README **and** the page in the same commit. |

## The Pipeline

```
0 BOOTSTRAP   repo? tenant? MCP? identity? version? publication?  → .gitignore armed, docs referenced by URL
1 RESEARCH    developer.dynatrace.com, depth-first       → evidence log with URLs actually read
2 SPEC        specs/<slug>.md                            → ⛔ USER APPROVAL GATE
3 BUILD       Strato-only, verified APIs                 → every symbol traced to a .d.ts or a doc page
4 VALIDATE    DQL on live tenant + data-shape assert     → real records shown to you
              + lint + security & quality review
5 RUN/DEPLOY  dt-app dev → dt-app deploy                 → ⛔ USER APPROVAL GATE before deploy
6 PUBLISH     bump version → CHANGELOG → README + page → ⛔ USER APPROVAL GATE before push
              → sanitize → commit → push → tag
```

## What It Prevents

| Symptom | Root cause | Rule |
| --- | --- | --- |
| App looks like a generic React dashboard | Third-party UI kit, or hand-rolled CSS colours | R3 |
| `Property 'xyz' does not exist on type` | Prop invented from memory | R2 |
| Tenant URL or token in a public repo | No sanitize pass before commit | R1 |
| Chart renders empty, with no error | Grail records handed to a component expecting a different shape | R7 |
| Query returns nothing in production | DQL never executed against a real tenant | R6 |
| Five line charts on one page | Visualization chosen by habit | R8 |
| "Co-Authored-By: Claude" in the history | Default commit trailer not stripped | R10 |
| Three deploys all reporting `1.0.0` | Version not bumped per publication | R12 |
| README describing the app as it was two features ago | Docs not updated in the same commit | R12 |
| DQL syntax that was valid last month | A vendored copy of the Dynatrace skills went stale | Phase 0 |
| Surprise DPS bill | Auto-refresh × tabs × wide timeframe, never measured | R6, README standard |

## Important Behaviors

### It is opinionated on purpose
The rules are stated as absolutes because "prefer Strato where reasonable" is, in practice,
"use Strato until something is mildly inconvenient". If you disagree with a rule, edit the
`SKILL.md` — it is Markdown, it is yours.

### It will refuse to guess
When a package is not installed and the docs are unreachable, the skill's instruction is to
say *"I cannot verify this"* and offer the two actions that would unblock it, rather than
emit a plausible-looking prop. Expect more questions and fewer silent failures.

### The Strato inventory in the references is a snapshot, not a contract
It was verified against `@dynatrace/strato-components@3.11.3` in August 2026. The skill's
own instruction is to re-verify in your project's `node_modules` — the inventory is a map to
speed up the lookup, never a substitute for it.

### `strato-components-preview` is deprecated
Most training data and most older `AGENTS.md` files say charts and tables live in
`@dynatrace/strato-components-preview`. As of Strato 3.x that package is deprecated and
re-exports `@dynatrace/strato-components`. The skill corrects this explicitly, because it is
the single most common source of outdated Dynatrace App code.

### Nothing is vendored — the docs are read live
The skill carries no copy of `developer.dynatrace.com`, `docs.dynatrace.com`, or
Dynatrace's agent skills. It carries their **URLs**, and instructs the agent to open them
during the session. That is why it works better in your VS Code than in a sandbox with no
network: the references are live, not frozen. Every URL you were given for a Dynatrace App
— design, components, data visualizations, icons, patterns, foundations, AppEngine, the
develop guides, security and code optimization — is in
[`documentation-research.md`](skills/development-pattern-for-dynatrace/references/documentation-research.md).

### The secret scanner is a net, not a guarantee
`scan-secrets.sh` catches known patterns — Dynatrace tokens, tenant URLs and IDs, entity
IDs, private keys, emails, hardcoded credentials. It cannot catch a secret that looks like
ordinary text. A clean scan is necessary, not sufficient: read your own diff.

### It treats the version and the docs as part of the change
Not as follow-up work. A commit that changes behaviour without bumping `app.version` and
updating the README and the page is, under this pattern, an incomplete commit. The bump
level is decided by one question most people get wrong: **adding an OAuth scope is a MAJOR
change**, because every existing user is pushed through a consent screen on next load —
even when the code diff is three lines.

### It has opinions about your bill
Cost is treated as a first-class design constraint, not an afterthought — the spec template
has a cost section, the review checklist has cost items, and the README standard requires a
DPS section on every app you publish.

## Cost & Consumption (DPS)

> ⚠️ **The skill itself costs nothing. The apps it helps you build do.**

Every Grail query consumes Dynatrace Platform Subscription budget, billed on **data analyzed
(GB scanned)**. This skill's job is to make that visible before it appears on an invoice —
but the apps you build are yours, and so is the bill.

The pattern requires four things of every app:

| Requirement | Where it is enforced |
| --- | --- |
| A cost section in the spec, before the code exists | `spec.template.md` |
| Filter-early, project-only-needed-fields queries | `dql-and-mcp.md` |
| Auto-refresh off by default, or a defensibly slow interval | `code-review-checklist.md` |
| A **Query Cost & Consumption (DPS)** section in the README and on the page | `readme-and-page.md` |

The estimate that goes in every app README:

```
GB/month ≈ GB_per_query × queries_per_hour × hours_per_day × days × concurrent_users
cost     ≈ GB/month × your DPS "Grail Query – data analyzed" rate
```

| Scenario | Queries/month | GB scanned* | Est. cost** |
| --- | --- | --- | --- |
| Auto-refresh **OFF**, 5 users, manual use | ~2,000 | ~2 GB | Negligible |
| Auto-refresh **ON** (1 min), 5 users, 8h/day | ~53,000 | ~260 GB | Low–moderate |
| Auto-refresh **ON** (1 min), 20 users, 8h/day | ~211,000 | ~1 TB | ⚠️ Watch this |

\* Assuming ~5 MB scanned per query in a busy environment; small environments scan well under
1 MB. \*\* Multiply GB scanned by your contract's *"Grail Query – data analyzed"* rate — the
dollar amount depends entirely on your rate card.

**Measure, don't estimate.** `queryExecute(...)` returns `metadata.grail.scannedBytes` and
`scannedRecords` — the real cost per execution. Aggregates live in
**Account Management → Cost & usage → Grail Query**.

## Repository Structure

```
skills/development-pattern-for-dynatrace/
├── SKILL.md                              # the twelve rules, the pipeline, routing
├── references/
│   ├── security-and-secrets.md           # R1 — gitignore, placeholders, scanning, rotation
│   ├── verification-protocol.md          # R2 — source hierarchy, commands, evidence log
│   ├── documentation-research.md         # R4 — the doc map and the depth protocol
│   ├── strato-components.md              # R3 — package map, imports, component inventory
│   ├── strato-dataviz.md                 # R7, R8 — chart choice, data contracts, shape checks
│   ├── dql-and-mcp.md                    # R6 — MCP setup, validation loop, DQL traps, cost
│   ├── spec-driven-workflow.md           # R5 — phases, gates, spec anatomy
│   ├── app-lifecycle.md                  # dt-app commands, scopes, deploy, troubleshooting
│   ├── code-review-checklist.md          # R9 — the pre-push review
│   ├── git-and-publishing.md             # R1, R10, R11 — commit, push, publish
│   ├── release-and-docs-sync.md          # R12 — SemVer, CHANGELOG, docs sync matrix
│   └── readme-and-page.md                # the README + GitHub Pages standard
├── assets/
│   ├── templates/                        # gitignore, eslint, app.config, spec, README, page, AGENTS, hook
│   └── scripts/scan-secrets.sh           # the pre-commit scanner
docs/index.html                           # source for this project's page
.github/workflows/pages.yml               # publishes docs/ to the generated gh-pages branch
```

`gh-pages` is generated and force-pushed by CI — edit `docs/`, never that branch.

## Templates & Tools

| File | Use |
| --- | --- |
| [`gitignore.template`](skills/development-pattern-for-dynatrace/assets/templates/gitignore.template) | Dynatrace-aware `.gitignore` — includes `.dt-app/`, which caches OAuth tokens |
| [`eslint.config.mjs.template`](skills/development-pattern-for-dynatrace/assets/templates/eslint.config.mjs.template) | Security + SDL + no-secrets rules, and a hard block on non-Strato UI libraries |
| [`app.config.json.template`](skills/development-pattern-for-dynatrace/assets/templates/app.config.json.template) | Placeholder-safe app config |
| [`env.example.template`](skills/development-pattern-for-dynatrace/assets/templates/env.example.template) | The committed half of the `.env` pair |
| [`spec.template.md`](skills/development-pattern-for-dynatrace/assets/templates/spec.template.md) | One-page spec with data contract, visualization rationale, cost, docs impact and evidence log |
| [`CHANGELOG.template.md`](skills/development-pattern-for-dynatrace/assets/templates/CHANGELOG.template.md) | Keep a Changelog format, with the scope/cost callouts this pattern requires |
| [`README.template.md`](skills/development-pattern-for-dynatrace/assets/templates/README.template.md) | The app README standard, disclaimer and DPS section included |
| [`page.template.html`](skills/development-pattern-for-dynatrace/assets/templates/page.template.html) | Self-contained GitHub Pages site — no build step, no CDN |
| [`AGENTS.template.md`](skills/development-pattern-for-dynatrace/assets/templates/AGENTS.template.md) | Drop into an app repo so future sessions inherit the pattern |
| [`pre-commit.template`](skills/development-pattern-for-dynatrace/assets/templates/pre-commit.template) | Git hook that blocks a commit containing secrets |
| [`scan-secrets.sh`](skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh) | The scanner itself |

Run the scanner directly, any time:

```bash
bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh
git diff --cached | bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh --stdin
```

## Versioning

This skill follows [Semantic Versioning](https://semver.org/) — the same standard R12
requires of the apps it helps you build. "Breaking" means breaking for the person *using*
the skill: a rule renumbered, a reference removed, a template's contract changed.

- `.claude-plugin/plugin.json` → `version` is the single source of truth.
- Every release is recorded in [CHANGELOG.md](CHANGELOG.md) and tagged `v<version>`.

**Current version: 1.1.0** — adds R12, and stops vendoring the Dynatrace docs.

## Keeping It Current

Strato ships breaking changes roughly quarterly, and the Dynatrace docs move. Two habits
keep the skill honest:

- **Re-verify the inventory** when `dt-app update` bumps `@dynatrace/*`:
  `grep -oE "export \{[^}]*\}" node_modules/@dynatrace/strato-components/charts/index.d.ts`
- **Record traps in your app's `AGENTS.md`** as you hit them, so the next session starts
  ahead instead of level.
- **Nothing to sync from Dynatrace.** The docs and the official skills are referenced by
  URL and fetched per session, so they cannot go stale here — that is the point.

Corrections and additions are welcome — open an issue or a pull request.

---

Related: [Command Center for Dynatrace](https://github.com/adrianorafael/Command-Center-for-Dynatrace) —
a demonstration app built following this pattern.

Learn more about the platform at [Dynatrace Developer](https://developer.dynatrace.com/).
