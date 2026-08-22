---
name: development-pattern-for-dynatrace
description: "Standard for building native Dynatrace Apps (AppEngine, dt-app, Strato Design System) with an AI coding agent. Load BEFORE writing, reviewing, running, deploying or publishing any Dynatrace App code. Enforces: no invented components/props/DQL (verify against node_modules .d.ts and developer.dynatrace.com); Strato-only UI (no MUI/Tailwind/Recharts/Chart.js/d3); depth-first doc research; spec-driven development with approval gates; DQL executed against a live tenant via the Dynatrace MCP server; verified data shapes before feeding charts or tables; varied, fit-for-purpose visualizations instead of all-line-chart screens; a security review of AI-written code; secret-safe publishing (no tokens, .env, tenant IDs or URLs — placeholders instead); README + GitHub Pages house style; no AI co-authorship in commits; and the '<App Name> for Dynatrace' naming convention. Trigger on: 'Dynatrace app', 'dt-app', 'AppEngine', 'Strato', 'DQL', 'Grail', 'app.config.json', 'dynatrace deploy', 'vibecoding a Dynatrace app'."
license: MIT
---

# Development Pattern for Dynatrace

A guardrail skill for building **native Dynatrace Apps** with an AI coding agent.

Its job is not to make you faster at guessing. Its job is to make guessing **impossible**:
every component, prop, DQL function, scope and CLI flag you emit must be traceable to a
source you actually read in this session.

---

## R — The eleven non-negotiable rules

These override any other instinct, including "this is a tiny change".

| # | Rule | Enforced by |
| --- | --- | --- |
| **R1** | **Never publish a secret.** No tokens, `.env`, tenant IDs, tenant URLs, emails, internal hostnames, customer names or screenshots with real data reach a commit. Sanitize to placeholders first. | [security-and-secrets.md](references/security-and-secrets.md) |
| **R2** | **Never invent an API.** A Strato component, prop, icon, hook, SDK client, DQL function or `dt-app` flag may only be used after it is verified in `node_modules/**/*.d.ts` or on `developer.dynatrace.com`. | [verification-protocol.md](references/verification-protocol.md) |
| **R3** | **Strato only.** The UI is built exclusively from `@dynatrace/strato-*`. Third-party UI kits and chart libraries are forbidden. | [strato-components.md](references/strato-components.md) |
| **R4** | **Research before spec.** Read the relevant `developer.dynatrace.com` section *and follow its in-page links* before writing a spec. Never answer from memory alone. | [documentation-research.md](references/documentation-research.md) |
| **R5** | **Spec before code.** Non-trivial work gets a `specs/<slug>.md` and an explicit user approval gate. | [spec-driven-workflow.md](references/spec-driven-workflow.md) |
| **R6** | **Every DQL is executed before it ships.** Validate against a live tenant through the Dynatrace MCP server and show the user the real returned shape. | [dql-and-mcp.md](references/dql-and-mcp.md) |
| **R7** | **Every visualization is fed a verified data shape.** Convert, then assert the contract the component expects, then render. Never hand a raw Grail record set to a chart and hope. | [strato-dataviz.md](references/strato-dataviz.md) |
| **R8** | **No monotonous screens.** Pick the visualization from the data's shape and question, not from habit. A screen of five line charts is a defect. | [strato-dataviz.md](references/strato-dataviz.md) |
| **R9** | **Review AI-written code like hostile code.** Run the security + quality checklist before every push. | [code-review-checklist.md](references/code-review-checklist.md) |
| **R10** | **No AI co-authorship.** No `Co-Authored-By: Claude`, no "Generated with …", no AI attribution in commits, PRs, README, page or code comments — unless the user explicitly asks for it. | [git-and-publishing.md](references/git-and-publishing.md) |
| **R11** | **Naming convention `<App Name> for Dynatrace`.** Repository, README `# H1`, page title and Hub listing all use it. | [git-and-publishing.md](references/git-and-publishing.md) |

If a rule cannot be satisfied, **stop and say so**. Do not ship a degraded version silently.

---

## Phase 0 — Session bootstrap (run once, before anything else)

Do not start coding until these five answers exist. Ask them in one batch; do not ask
one at a time.

1. **Repository** — "Is there a Git repository for this app? URL, or should I initialize one?"
   → Store the answer. Every later "push?" moment uses it. (R1, R10, R11)
2. **Target tenant** — "Which Dynatrace environment will this run against?"
   → Keep it **in memory and in `.env` only**. It never enters a tracked file. (R1)
3. **MCP** — "Is the Dynatrace MCP server connected?" Detect first, ask second.
   → Without it, DQL cannot be validated (R6). See [dql-and-mcp.md](references/dql-and-mcp.md).
4. **App identity** — app display name, `app.id` (must start with `my.` for unsigned apps), icon.
5. **Publication intent** — will this become a public repo + GitHub Pages? If yes, the
   README/page standard applies from day one. See [readme-and-page.md](references/readme-and-page.md).

Then, in the same phase:

- **Install the official Dynatrace knowledge locally.** Run
  `npx skills add dynatrace/dynatrace-for-ai`, or clone to a gitignored directory.
  These are the authoritative `.md` sources for DQL, Grail semantics, problems, logs, costs
  and the JS runtime. Prefer them over your own recollection.
  → [dql-and-mcp.md § Official Dynatrace skills](references/dql-and-mcp.md)
- **Establish the secret baseline** before the first commit: `.gitignore`, placeholder
  sweep, secret scan. → [security-and-secrets.md](references/security-and-secrets.md)

---

## Phase 1 → 6 — The pipeline

```
0 BOOTSTRAP   repo? tenant? MCP? identity? publication?  → official skills installed, .gitignore armed
1 RESEARCH    developer.dynatrace.com, depth-first       → evidence log with URLs actually read
2 SPEC        specs/<slug>.md                            → ⛔ USER APPROVAL GATE
3 BUILD       Strato-only, verified APIs                 → every symbol traced to a .d.ts or a doc page
4 VALIDATE    DQL on live tenant + data-shape assert     → real records shown to the user
              + lint + security & quality review
5 RUN/DEPLOY  dt-app dev → dt-app deploy                 → ⛔ USER APPROVAL GATE before deploy
6 PUBLISH     sanitize → commit → push → README + page   → ⛔ USER APPROVAL GATE before push
```

Three gates are hard stops. Never cross them on your own initiative.

Full definitions: [spec-driven-workflow.md](references/spec-driven-workflow.md).

---

## Reference routing

Load the reference **before** doing the work, not after it fails.

| You are about to… | Read first |
| --- | --- |
| Commit, push, create a repo, write a commit message, name a repo | [git-and-publishing.md](references/git-and-publishing.md) |
| Touch `app.config.json`, `.env`, `launch.json`, or anything with a tenant URL/token | [security-and-secrets.md](references/security-and-secrets.md) |
| Use any Strato component, prop, icon or design token | [strato-components.md](references/strato-components.md) |
| Put data into a chart, table, tile or map; choose a visualization | [strato-dataviz.md](references/strato-dataviz.md) |
| Write, run, or optimize DQL; set up / use the MCP server | [dql-and-mcp.md](references/dql-and-mcp.md) |
| Look something up in the Dynatrace docs | [documentation-research.md](references/documentation-research.md) |
| Claim that an API exists | [verification-protocol.md](references/verification-protocol.md) |
| Write a spec, or decide whether work needs one | [spec-driven-workflow.md](references/spec-driven-workflow.md) |
| Run locally, deploy, add scopes, add a function/action/intent | [app-lifecycle.md](references/app-lifecycle.md) |
| Review code before a push | [code-review-checklist.md](references/code-review-checklist.md) |
| Write the README or the GitHub Pages site | [readme-and-page.md](references/readme-and-page.md) |

Templates ready to copy: [`assets/templates/`](assets/templates/).
Secret scanner: [`assets/scripts/scan-secrets.sh`](assets/scripts/scan-secrets.sh).

---

## Anti-hallucination: the four moves

**1. Verify, don't recall.** Before you write `<HoneycombChart>`, prove it exists:

```bash
grep -rn "HoneycombChart" node_modules/@dynatrace/strato-components/charts/index.d.ts
```

Before you write a prop, read the component's own `.d.ts`:

```bash
find node_modules/@dynatrace/strato-components/charts/honeycomb -name '*.d.ts'
```

**2. Say "I don't know" out loud.** If the docs are unreachable and the package is not
installed, the honest output is: *"I cannot verify `X` — install the package or give me
the doc page, and I will."* An invented prop that silently no-ops is worse than a question.

**3. Cite what you read.** Each spec carries an evidence log: URL or file path, and the
one line it justified. A claim with no source in the log does not go in the spec.

**4. Prefer the smallest confirmed API.** When two shapes look plausible, use the one the
`.d.ts` proves, even if the other looks nicer.

---

## Version reality check (verified 2026-08)

Facts that older training data and older `AGENTS.md` files get wrong. Re-verify each in
the project's own `node_modules` — these are the *current* answers, not permanent ones.

| Claim | Reality |
| --- | --- |
| "Most components live in `@dynatrace/strato-components-preview`" | **Outdated.** As of Strato 3.x, `strato-components-preview` is **deprecated** and simply re-exports `@dynatrace/strato-components`. Charts, tables, filters, forms and editors all live in `@dynatrace/strato-components` now. |
| "Import from the package root" | **Wrong, always.** Import from the subpath: `@dynatrace/strato-components/charts`. Root imports break tree-shaking and are blocked by the project's ESLint rule. |
| "`@dynatrace/strato-components-preview` is where charts are" | Charts subpath is `@dynatrace/strato-components/charts`. The preview package still resolves, but new code should not use it. |
| "The app can `fetch()` any external URL from the browser" | **No.** The `connect-src` CSP directive is not configurable for apps. External calls go through an **app function** (server-side JS runtime). |
| "Store the token in the app" | **No.** Use the Credential Vault, or an app function with `environment-api:credentials:read`. |

---

## Failure modes this skill exists to prevent

| Symptom | Root cause | Rule |
| --- | --- | --- |
| App looks like a generic React dashboard, not Dynatrace | Third-party UI kit, or hand-rolled CSS colors | R3 |
| `Property 'xyz' does not exist on type` after "it should work" | Prop invented from memory | R2 |
| Tenant URL / token pushed to a public repo | No sanitize pass before commit | R1 |
| Chart renders empty with no error | Grail records handed to a component expecting a different shape | R7 |
| Query returns nothing in production but "looked right" | DQL never executed against a real tenant | R6 |
| Five line charts on one page | Visualization chosen by habit | R8 |
| Reviewer sees "Co-Authored-By: Claude" | Default commit trailer not stripped | R10 |
| Surprise DPS bill | Auto-refresh × tabs × wide timeframe, never measured | [readme-and-page.md § cost](references/readme-and-page.md) |

---

## Talking to the user

- Report what you **verified**, not what you assumed. "Confirmed in `charts/index.d.ts`" beats "should work".
- Before any push, show the **sanitize diff**: what was replaced with a placeholder.
- Before any deploy, name the **target tenant** and wait for a yes.
- When you decline to guess, offer the concrete next step that would unblock you.
