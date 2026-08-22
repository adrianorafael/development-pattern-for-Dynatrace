# README & project page — the house standard

Every app published under this pattern ships two documents that say the same things at
different depths: a **README.md** (for someone about to run it) and a **GitHub Pages site**
under `docs/` (for someone deciding whether to care).

They exist to do four jobs, and the first three are about protecting the author:

1. **Disclaim.** Not official, not affiliated, not supported by Dynatrace.
2. **Disclaim cost.** No responsibility for resource and license consumption.
3. **Quantify cost anyway.** An honest DPS estimate, with the method to measure it properly.
4. **Enable.** Someone can clone it and run it against their own tenant without asking you.

Reference implementation: [Command Center for Dynatrace](https://github.com/adrianorafael/Command-Center-for-Dynatrace)
and its [project page](https://adrianorafael.github.io/Command-Center-for-Dynatrace/).

---

## The disclaimer — non-negotiable, verbatim in spirit

Immediately after the intro, before anything technical:

```markdown
> ⚠️ **Disclaimer**
>
> This app is provided by the developer with **no affiliation with Dynatrace** and
> **no responsibility** for any failures, issues, or consumption of resources and
> licenses. It is a **study version with no official support**.
>
> Deciding to download and install it in your environment is **at the user's own risk**.
> You are equally free to improve and expand its features.
```

And in the page footer:

```
Not affiliated with, endorsed by, or supported by Dynatrace. Provided "as is" for study
and demonstration purposes. Dynatrace and the Dynatrace logo are trademarks of Dynatrace LLC.
```

Say plainly what the app **is**: a demonstration app showing how to build a Dynatrace App
following `developer.dynatrace.com` and Strato — and, when true, that it is **not finished**.
Understating maturity is safe; overstating it is not.

---

## README structure

Template: [`../assets/templates/README.template.md`](../assets/templates/README.template.md).

```markdown
# <App Name> for Dynatrace                      ← R11

🌐 **Project page:** https://<user>.github.io/<App-Name>-for-Dynatrace/

<one paragraph: what it is, who maintains it, what it demonstrates>

> ⚠️ **Disclaimer** …                           ← above, verbatim

## Overview
## Table of Contents
## Prerequisites
## Installation and Configuration               ← incl. the YOUR-ENVIRONMENT placeholder step
## Running in Development Mode
## Publishing to Environment (Deploy)
## Publishing to Another Environment            ← incl. what does NOT migrate
## Required OAuth Scopes                        ← table: scope → why
## App Features                                 ← one subsection per feature
## Important Behaviors                          ← the surprising truths
## Query Cost & Consumption (DPS)               ← the section people skip and shouldn't
## Relevant File Structure
## Available Scripts
```

### Sections that carry the weight

**Installation and Configuration** — must contain the placeholder-substitution step,
explicitly:

> Replace `YOUR-ENVIRONMENT` with your own Dynatrace tenant ID. The environment URL has the
> form `https://<tenant-id>.apps.dynatrace.com/` — copy it from the address bar of your
> Dynatrace environment.

List **every** file carrying a placeholder (`app.config.json`, `.vscode/launch.json`,
`.env.example`). A placeholder the reader doesn't know about is a broken setup.

Also state: **`app.id` must start with `my.`** for unsigned apps.

**Publishing to Another Environment** — the warnings, not just the commands: App State is
per-tenant and does not migrate; the target needs the same Grail tables; the target's admin
must grant the scopes.

**Required OAuth Scopes** — a table of scope → the concrete feature that needs it, plus the
note that adding scopes post-release forces users to re-consent.

**Important Behaviors** — where you are honest about the things a reader would otherwise
discover at 2am: platform limitations you worked around, state that is shared across users,
last-write-wins semantics, size caps, what happens without Grail, known unfinished parts.

---

## The cost section — write it, every time

This is the section that protects the author *and* the reader. Structure:

**1. A warning header.**

> ⚠️ **Read this before running the app in a production environment.** This app runs live
> DQL queries against Grail, which consumes Dynatrace Platform Subscription (DPS) budget.
> Understand and measure it to avoid surprises on your bill.

**2. What drives cost** — name the actual query, link the file, and rank the multipliers:

| Factor | Effect | Notes |
| --- | --- | --- |
| **Auto-refresh (1 min)** | 🔴 **Dominant** — 1 query per minute, per open tab | On by default. Biggest lever. |
| **Timeframe width** | Wider window = more data scanned | Default is "Today". |
| **Concurrent users / tabs** | Multiplies linearly | Each instance polls independently. |
| **Data volume** | More records = more GB scanned | Environment-dependent. |

**3. A formula and an illustrative scenario table** — clearly marked as illustrative:

```
GB/month ≈ GB_per_query × queries_per_hour × hours_per_day × days × concurrent_users
cost     ≈ GB/month × your DPS "Grail Query – data analyzed" rate
```

| Scenario | Queries/month | GB scanned* | Est. cost** |
| --- | --- | --- | --- |
| Auto-refresh **OFF**, 5 users, manual | ~2,000 | ~2 GB | Negligible |
| Auto-refresh **ON**, 5 users, 8h/day | ~53,000 | ~260 GB | Low–moderate |
| Auto-refresh **ON**, 20 users, 8h/day | ~211,000 | ~1 TB | ⚠️ Watch this |

> \* Assuming ~5 MB scanned per query in a busy environment; small environments scan well
> under 1 MB. \*\* Multiply GB scanned by your contract's *"Grail Query – data analyzed"*
> rate — the dollar amount depends entirely on your rate card, so **measure before relying
> on estimates**.

**Never print a currency figure.** You do not know their rate card. Give GB and the
multiplication step.

**4. How to measure precisely** — because the estimate is not the point:

- Run the query in a **Notebook** and read the scan metadata; programmatically,
  `queryExecute(...)` returns `metadata.grail.scannedBytes` and `scannedRecords`.
- Confirm aggregates in **Account Management → Cost & usage → Grail Query** after running
  on/off scenarios.

**5. How to reduce it** — concrete, with file references: raise the refresh interval or
default it off, keep the timeframe narrow, project only needed fields with `| fields …`
(Grail is columnar), avoid many open tabs.

**6. The query itself**, verbatim, so readers can price it themselves.

---

## The GitHub Pages site

`docs/index.html` — a **single self-contained HTML file**. No build step, no CDN, no
external JS. Template:
[`../assets/templates/page.template.html`](../assets/templates/page.template.html).

### Turning Pages on

The simplest route, if the owner is at a keyboard: *Settings → Pages → Source: Deploy from
a branch → `main` + `/docs` → Save.* Site lands at `https://<user>.github.io/<Repo-Name>/`.

**An agent cannot do this.** Creating a Pages site through the REST API
(`POST /repos/{owner}/{repo}/pages`) requires **admin** permission on the repository, which
`GITHUB_TOKEN` never has. `actions/configure-pages` with `enablement: true` therefore fails
with:

```
##[error]Create Pages site failed. Error: Resource not accessible by integration
```

Do not retry it, and do not tell the user it will work — it will not.

**What does work without any manual step:** pushing a `gh-pages` branch. GitHub
auto-enables Pages for a public repository the first time that branch appears, and serves
it from the branch root. So publish `docs/` *to* `gh-pages` in CI and keep `docs/` as the
single source of truth:

```yaml
permissions:
  contents: write        # NOT pages: write — this pushes a branch, it does not call the Pages API
# …
- run: |
    cp -r docs "$RUNNER_TEMP/site"
    touch "$RUNNER_TEMP/site/.nojekyll"     # plain HTML, no Jekyll preprocessing
    cd "$RUNNER_TEMP/site"
    git init -q -b gh-pages
    git config user.name  "github-actions[bot]"
    git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
    git add -A && git commit -q -m "Publish project page from ${GITHUB_SHA:0:7}"
    git push -f "https://x-access-token:${GH_TOKEN}@github.com/${GITHUB_REPOSITORY}.git" gh-pages
```

Never hand-edit `gh-pages` — it is generated and force-pushed. Two hand-maintained copies
of the same page is the fastest way to ship a site that contradicts its own README.

**One more owner-only trap:** pushing the first branch to an *empty* repository makes that
branch the default. If the first push went to a feature branch, the repo's default branch
is now that feature branch — only the owner can change it, under
*Settings → General → Default branch*. Tell the user; do not leave it silently wrong.

### Section order

| # | Section | Contains |
| --- | --- | --- |
| — | **Sticky nav** | brand + anchors + "GitHub ↗" |
| 1 | **Hero** | eyebrow badge, `<App Name> for Dynatrace` with a gradient word, one-sentence subtitle, two CTAs, 3–4 stat cards |
| 2 | **Disclaimer banner** | amber, directly under the hero, above the fold |
| 3 | **Features** | card grid, one card per feature: icon, title, description, `meta` footnote |
| 4 | **How it works** | left-border callouts for the surprising behaviours |
| 5 | **Tech stack** | chip row: React, TypeScript, Strato, DQL/Grail, App State, dt-app, app id |
| 6 | **Query cost & consumption** | the drivers table, the scenario table, measure/reduce columns, the query |
| 7 | **Project structure** | annotated tree in a `<pre>` |
| 8 | **Quick start** | prerequisites, npm scripts, a copy-pasteable block, required scopes |
| 9 | **Troubleshooting** | symptom → cause & fix table |
| — | **Footer** | author, source, Dynatrace Developer link, trademark disclaimer |

### Visual conventions

Dark, Dynatrace-adjacent, self-contained:

```css
--bg:#0b0b16; --bg-elev:#12121f; --bg-card:#16162a; --border:#262640;
--text:#e7e7f0; --text-dim:#a0a0b8; --text-faint:#6b6b85;
--blue:#1496ff; --green:#b4dc00; --purple:#8a4fd0;
--critical:#ff6b6b; --warning:#ffc043;
--radius:14px; --maxw:1080px;
```

- Gradient accent on one hero word: `linear-gradient(120deg, var(--blue), var(--green) 55%, var(--purple))`.
- Cards lift on hover (`translateY(-3px)`), sticky nav with `backdrop-filter: blur(10px)`.
- Tables scroll inside `overflow-x:auto` — the body never scrolls sideways.
- Responsive: `repeat(auto-fit, minmax(260px, 1fr))` grids; nav links collapse under 860px.
- `<link rel="icon" href="icon.svg">` beside `index.html`.
- Set `<meta name="description">` — it is the search and social preview.

**These hex values are for the marketing page only.** They have nothing to do with the app:
inside the app it is design tokens, always (R3).

### README ↔ page consistency (R12)

The page is not a rewrite. Same disclaimer, same scopes, same cost numbers, same
troubleshooting rows, **same version**. When one changes, both change **in the same
commit** — together with the version bump and the CHANGELOG entry. Divergence between them
is the most common rot in this pattern.

Which change forces which update, and at which SemVer level:
→ [release-and-docs-sync.md](release-and-docs-sync.md)

---

## Applying this to a skill instead of an app

Same skeleton, different nouns:

| App | Skill |
| --- | --- |
| Prerequisites (Node, Grail) | Which agents support it |
| Installation & Configuration | `npx skills add …` / plugin / manual copy |
| Running in Development Mode | Invoking or triggering the skill |
| Deploy | Not applicable — drop it |
| OAuth scopes | Tooling the skill expects (MCP, dtctl) |
| App features | What the skill enforces, rule by rule |
| Query cost | Cost of what the skill *makes you build* — keep the section, reframe it |
| File structure | `SKILL.md` + `references/` map |

The disclaimer stays exactly as it is. A skill that shapes production apps carries the same
"no affiliation, no responsibility, your own risk" caveat as the apps it produces.
