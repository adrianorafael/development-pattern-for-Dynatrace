<!--
  README template — "Development Pattern for Dynatrace".
  Replace every <PLACEHOLDER>. Delete sections that genuinely do not apply,
  but NEVER delete the Disclaimer or the Query Cost & Consumption section.
-->

# <App Name> for Dynatrace

🌐 **Project page:** https://<user>.github.io/<App-Name>-for-Dynatrace/

**<App Name> for Dynatrace** is an app created and maintained by [@<user>](https://github.com/<user>) on GitHub.

<One paragraph: what it does and who it is for.>

This app is **not finished** — it is a **demonstration app** meant to show how easy it is
to build your own Dynatrace App through **vibecoding**, following the official documentation
at [developer.dynatrace.com](https://developer.dynatrace.com/) and using **Strato Design**
to style it with Dynatrace's Design System.

> ⚠️ **Disclaimer**
>
> This app is provided by the developer with **no affiliation with Dynatrace** and **no responsibility** for any failures, issues, or consumption of resources and licenses. It is a **study version with no official support**.
>
> Deciding to download and install it in your environment is **at the user's own risk**. You are equally free to improve and expand its features.

## Overview

<Two or three sentences: architecture, main capability, what it demonstrates.>

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation and Configuration](#installation-and-configuration)
3. [Running in Development Mode](#running-in-development-mode)
4. [Publishing to Environment (Deploy)](#publishing-to-environment-deploy)
5. [Publishing to Another Environment](#publishing-to-another-environment)
6. [Required OAuth Scopes](#required-oauth-scopes)
7. [App Features](#app-features)
8. [Important Behaviors](#important-behaviors)
9. [Query Cost & Consumption (DPS)](#query-cost--consumption-dps)
10. [Relevant File Structure](#relevant-file-structure)
11. [Available Scripts](#available-scripts)
12. [Versioning & Changelog](#versioning--changelog)

## Prerequisites

- Node.js >= 18 (recommended: the version pinned by `dt-app`)
- npm >= 9
- Dynatrace account with Grail enabled (`<tables used>`)
- Access to the environment via the `dt-app` CLI

```bash
npm install
```

## Installation and Configuration

### Step 1 — Configure the target environment in `app.config.json`

```json
{
  "environmentUrl": "https://YOUR-ENVIRONMENT.apps.dynatrace.com/",
  "app": { "id": "my.<app.id>" }
}
```

Replace `YOUR-ENVIRONMENT` with your own Dynatrace tenant ID. The environment URL has the
form `https://<tenant-id>.apps.dynatrace.com/` — copy it from the address bar of your
Dynatrace environment.

**IMPORTANT:** the app ID must start with `my.` for unsigned apps. IDs without that prefix
require digital app signing.

### Step 1b (optional) — Update the debug URL in `.vscode/launch.json`

This file ships with a `YOUR-ENVIRONMENT` placeholder and is used only for in-IDE
debugging — it does not affect `dt-app dev` / `deploy`.

### Step 2 — Authenticate

```bash
npx dt-app auth
```

Opens your browser for Dynatrace login. On first run you will be asked to grant the scopes
listed in [Required OAuth Scopes](#required-oauth-scopes). An "invalid scopes" error means
the tenant or your user lacks the corresponding IAM grants.

## Running in Development Mode

```bash
npx dt-app dev
```

**Open the link printed in the terminal, not `localhost:3000` directly** — the app must run
inside the Dynatrace context. Changes hot-reload.

## Publishing to Environment (Deploy)

```bash
npx dt-app deploy
```

Builds and publishes to the environment in `app.config.json`. Afterwards the app is
available to all tenant users under **Dynatrace → Apps → <App Name>**.

## Publishing to Another Environment

Deployment is per environment.

1. Edit `environmentUrl` in `app.config.json`
2. `npx dt-app auth`
3. `npx dt-app deploy`

**ATTENTION:**
- App State is isolated per tenant — persisted data does **not** migrate.
- The target environment needs the same Grail tables, or views render empty.
- The target's administrator must grant users the declared scopes.

## Required OAuth Scopes

| Scope | Purpose |
| --- | --- |
| `<scope>` | `<the concrete feature that needs it>` |

Adding a scope after deployment requires every existing user to re-authorize on next load.

## App Features

### <Feature>
- <what the user sees and can do>

## Important Behaviors

### <Behaviour a reader would otherwise discover at 2am>
<Platform limitations, shared state semantics, size caps, what happens without Grail.>

## Query Cost & Consumption (DPS)

> ⚠️ **Read this before running the app in a production environment.** This app runs live
> DQL queries against Grail, which consumes Dynatrace Platform Subscription (DPS) budget.
> Understand and measure it to avoid surprises on your bill.

### What drives cost

The meaningful cost driver is the **Grail query** in [`<file>`](<file>). Grail consumption
is billed by **data analyzed (GB scanned)** per execution.

| Factor | Effect on cost | Notes |
| --- | --- | --- |
| **Auto-refresh** | 🔴 **Dominant** — 1 query per interval, per open tab | `<default>` |
| **Timeframe width** | Wider window = more data scanned | Default `<...>` |
| **Concurrent users / tabs** | Multiplies linearly | Each instance polls independently |
| **Data volume** | More records = more GB scanned | Environment-dependent |

### Rough estimate

```
GB/month ≈ GB_per_query × queries_per_hour × hours_per_day × days × concurrent_users
cost     ≈ GB/month × your DPS "Grail Query – data analyzed" rate
```

| Scenario | Queries/month | GB scanned* | Est. cost** |
| --- | --- | --- | --- |
| Auto-refresh **OFF**, 5 users, manual | ~2,000 | ~2 GB | Negligible |
| Auto-refresh **ON**, 5 users, 8h/day | ~53,000 | ~260 GB | Low–moderate |
| Auto-refresh **ON**, 20 users, 8h/day | ~211,000 | ~1 TB | ⚠️ Watch this |

\* Assuming ~`<n>` MB scanned per query. \*\* Multiply GB scanned by your contract's
*"Grail Query – data analyzed"* rate — the dollar amount depends entirely on your rate card.

### How to measure precisely (recommended)

1. Paste the query into a Dynatrace **Notebook** and inspect the scan metadata.
   Programmatically, `queryExecute(...)` returns `metadata.grail.scannedBytes` and
   `scannedRecords` — the real cost per execution.
2. **Account Management → Cost & usage → Grail Query** confirms aggregated consumption
   after running on/off scenarios.

### How to reduce cost

- Increase the auto-refresh interval, or turn it off by default — biggest single win.
- Keep the default timeframe narrow.
- Project only the fields you need with `| fields …` — Grail is columnar.
- Avoid leaving many app tabs open simultaneously.

## Relevant File Structure

- **`app.config.json`** — environmentUrl, app id, OAuth scopes, icon.
- **`ui/app/pages/<Page>.tsx`** — <what it contains>
- **`ui/app/hooks/<useX>.ts`** — <the query it owns>
- **`ui/app/types/<x>.ts`** — shared domain types, single source of truth.

## Available Scripts

| Script | Runs | Description |
| --- | --- | --- |
| `npm run start` | `dt-app dev` | Development mode with hot reload |
| `npm run build` | `dt-app build` | Production build into `dist/` |
| `npm run deploy` | `dt-app deploy` | Build and deploy to the configured environment |
| `npm run uninstall` | `dt-app uninstall` | Remove the app from the environment |
| `npm run update` | `dt-app update` | Update `@dynatrace` packages and apply migrations |
| `npm run lint` | `eslint .` | Lint, including security and secret rules |

## Versioning & Changelog

This app follows [Semantic Versioning](https://semver.org/), where "breaking" means
breaking **for the person running the app**:

| Bump | When |
| --- | --- |
| **MAJOR** | A scope changed (everyone must re-consent) · persisted state format changed without migration · a feature was removed · a default changed in a way that raises cost |
| **MINOR** | New view, chart, filter or setting — nothing existing breaks |
| **PATCH** | Bug fix, copy, styling, dependency bump, query optimized with identical results |

- `app.config.json` → `app.version` is the single source of truth.
- Every release is recorded in [CHANGELOG.md](CHANGELOG.md).
- The same version is never published twice.

**Current version:** see [`app.config.json`](app.config.json).

To learn more about the Dynatrace Platform, see
[Dynatrace Developer](https://developer.dynatrace.com/).

---

Built following [Development Pattern for Dynatrace](https://github.com/adrianorafael/development-pattern-for-Dynatrace).
