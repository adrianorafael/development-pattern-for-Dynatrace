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

**Load it with your first prompt about the app, then just build.** The skill triggers on its
own when you mention a Dynatrace App, `dt-app`, Strato, AppEngine, DQL or Grail — or invoke
it by name.

```
"Build me a Dynatrace app that shows Kubernetes workload health by namespace."
```

That one sentence starts the whole pipeline. You do not have to ask for a spec, and you do
not have to remember to ask for a review at the end.

### Strict at the edges, free in the middle

This is the part that matters, and it is the opposite of what "twelve rules" sounds like:

| | What happens | Who drives |
| --- | --- | --- |
| **Front-load** | Bootstrap questions → the docs are read **live** → `specs/<slug>.md` → ⛔ your approval | The skill. No implementation code exists yet. |
| **Middle** | Build it. Iterate, change direction, throw work away. | **You.** This is the vibecoding, and the skill stays out of the way — it only holds the invariants: Strato only, verified APIs, no secrets. |
| **Back-load** | The finished code is verified: DQL executed, data shapes asserted, security and quality review, cost measured, version bumped, docs updated → ⛔ your approval → push | The skill. Every item, against the code that actually exists. |

The goal is not to make building slow. It is to make the **beginning deliberate** and the
**end verified**, so the fast part in between stays fast without quietly accruing debt.

### What a session actually looks like

**1 — You ask.**

> Build me a Dynatrace app that shows Kubernetes workload health by namespace.

**2 — Five questions, in one batch.** Repository? Target tenant? Is the MCP server
connected? App name, `app.id` and starting version? Will this be a public repo with a
GitHub Pages site? Every later decision refers back to these instead of re-asking.

**3 — The documentation is read, not recalled.** `developer.dynatrace.com` — design,
components, data visualizations, patterns, foundations — followed depth-first through
in-page links, plus `docs.dynatrace.com` for platform behaviour, plus Dynatrace's own agent
skills fetched fresh for DQL and Grail semantics. Everything lands in an evidence log:
claim → source → verified.

**4 — A one-page spec, then a stop.** `specs/k8s-workload-health.md`: the goal, the exact
DQL and whether it has been executed, the scopes and what justifies each, the Strato
components and **why that chart for this data**, the data contract with a real sample row,
the four states, the DPS cost estimate, and the evidence log.

> Here's the spec. Two decisions I need from you: whether unhealthy means `CrashLoopBackOff`
> only or any non-Running phase, and whether auto-refresh should default on. The DQL is
> validated — it returned 43 records over the last 24h. Approve and I'll build it.

If you answer *"just build it"*, that is a valid answer — it is recorded as approved without
review and the work proceeds. The gate is about consent, not ceremony.

**5 — Now the vibe coding.** Build, iterate, change your mind. The skill is not narrating
rules at you here. It is holding three lines you cannot cross without being told: it will
not import a non-Strato component, it will not use a prop it has not verified in
`node_modules`, and it will not put a tenant URL in a tracked file.

**6 — The finished code is verified.** `tsc`, ESLint with the security rules, `dt-app build`.
The DQL executed against your tenant with real records shown to you. The data shape printed
and asserted before it reaches a chart. The screen opened in a browser and actually looked
at. The monotony test. The security and quality checklist, read adversarially.

**7 — Version and docs, then push.** `app.config.json` → `app.version` bumped at the right
level, a CHANGELOG entry, README and project page updated to match — all in the same commit
as the code. Sanitize pass, secret scan, then it names the version and asks before pushing.

### Set it up once per repository

Drop [`AGENTS.template.md`](skills/development-pattern-for-dynatrace/assets/templates/AGENTS.template.md)
into your app repository as `AGENTS.md` so future sessions inherit the pattern — including
the traps you have already hit — instead of starting level.

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
              → sanitize → commit → push
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

- `.claude-plugin/plugin.json` → `version` is the single source of truth — a config file,
  the same way `app.config.json` → `app.version` is for an app.
- Every release is recorded in [CHANGELOG.md](CHANGELOG.md).

**Current version: 1.2.0** — documents how the skill is used across a session, and
states versioning as a config-file concern rather than a git-tag one.

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
