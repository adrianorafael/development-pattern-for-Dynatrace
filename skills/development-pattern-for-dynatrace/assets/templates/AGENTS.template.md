# AI Coding Agent Instructions — <App Name> for Dynatrace

## Follow the development pattern

This project is built under **[Development Pattern for Dynatrace](https://github.com/adrianorafael/development-pattern-for-Dynatrace)**.
Load that skill before writing, reviewing, running, deploying or publishing any code here.

Its twelve non-negotiable rules apply to every change in this repository, including
"tiny" ones:

1. Never publish a secret — no tokens, `.env`, tenant IDs or tenant URLs.
2. Never invent an API — verify against `node_modules/**/*.d.ts` or developer.dynatrace.com.
3. Strato only — no MUI, Tailwind, Recharts, Chart.js or D3.
4. Research the docs, depth-first, before writing a spec.
5. Spec before code, with an approval gate.
6. Every DQL is executed against a live tenant before it ships.
7. Every visualization is fed a verified data shape.
8. No monotonous screens — chart type follows the data's shape and question.
9. Review AI-written code like hostile code.
10. No AI co-authorship in commits, PRs, README or page.
11. Naming convention: `<App Name> for Dynatrace`.
12. Every publication bumps `app.version` (SemVer); every push that changes behaviour,
    setup or cost updates the README **and** the project page in the same commit.

## Project specifics

- **App id:** `my.<...>` (must keep the `my.` prefix — unsigned app)
- **Grail tables used:** `<...>`
- **Scopes:** see `app.config.json`; each one carries a justification comment
- **Persistence:** `<App State key / none>` — App State is capped at 400 KB per tenant
- **Auto-refresh default:** `<off / interval>` — the dominant DPS cost driver
- **Version:** `app.config.json` → `app.version` is the single source of truth; releases are
  tagged `v<version>` and recorded in `CHANGELOG.md`

## Traps already hit in this repository

<Record them here so the next session starts ahead instead of level.>

- `@dynatrace/strato-components-preview` is deprecated — import from
  `@dynatrace/strato-components/<subpath>`.
- `TimeseriesChart` needs `datapoints[].start` as a `Date`, not an ISO string.

## Dynatrace knowledge: reference, never vendor

Dynatrace's own agent skills (https://github.com/Dynatrace/dynatrace-for-ai) are the
authority for DQL and Grail semantics. **Fetch the file you need at the moment you need
it** — they change between sessions, and a stale copy looks authoritative while being
wrong. Never commit one into this repository.

## Commands

```bash
npm install
npx dt-app auth      # authenticate against the environment in app.config.json
npx dt-app dev       # local dev — open the printed link, NOT localhost:3000
npm run lint         # ESLint incl. security + no-secrets rules
npx tsc --noEmit
npx dt-app deploy    # ONLY with explicit approval, and name the target tenant first
```
