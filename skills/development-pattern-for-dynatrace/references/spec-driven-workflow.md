# Spec-driven workflow — decide before you type

> Rule **R5**. Non-trivial work gets a written spec and an explicit approval gate.

Vibecoding fails in a specific, predictable way: the agent starts writing before the shape
of the thing is decided, the user corrects the visible symptoms one at a time, and after
six rounds the app is a pile of patches with no design. A spec costs ten minutes and
replaces those six rounds with one.

---

## Does this need a spec?

| Needs a spec | Skip the spec |
| --- | --- |
| New app, new page, new route | Copy fix, label change |
| New data source or new DQL query | Spacing/colour token swap |
| New visualization or table | Adding a verified prop to an existing component |
| New scope, function, action, or intent | Renaming a local variable |
| Persistence, cross-user state | Adding an obvious loading state |
| Anything touching cost, auth, or secrets | Dependency patch bump |

**When in doubt, write the spec.** It is cheaper than the rework.
"Skip the spec" work still obeys R1, R2, R3 and R9 — there is no exemption from those.

---

## Phase 0 — Bootstrap

The five questions from `SKILL.md`, asked **in one batch**:

1. Repository — existing URL, or initialize a new one?
2. Target tenant — which environment? (kept out of tracked files)
3. MCP — connected? (detect first, then ask)
4. App identity — display name, `app.id` (`my.*` for unsigned), icon
5. Publication — public repo + GitHub Pages?

Then: install the official Dynatrace skills, arm `.gitignore`, run the first secret scan.

---

## Phase 1 — Research

Depth-first through `developer.dynatrace.com`, per
[documentation-research.md](documentation-research.md). Output: an evidence log with URLs
you actually read and `.d.ts` paths you actually opened.

Do not begin the spec while any load-bearing question is still unanswered. "I'll figure out
the chart type while coding" is how R8 gets violated.

---

## Phase 2 — Spec ⛔ GATE

Write `specs/<slug>.md` from
[`../assets/templates/spec.template.md`](../assets/templates/spec.template.md).
It is short — one page. Its sections:

| Section | The question it answers |
| --- | --- |
| **Goal** | What can the user do afterwards that they cannot do now? One sentence. |
| **Non-goals** | What is explicitly out of scope? Prevents silent scope creep. |
| **Data** | Which Grail tables/fields? The **exact DQL**, and its validation status. |
| **Scopes** | Which `app.config.json` scopes, and the concrete feature justifying each. |
| **UI** | Which Strato components, from which subpaths. Layout sketch. |
| **Visualization** | Which chart, **and why that one for this data** (R8). |
| **Data contract** | The exact shape passed to each component, with a real sample row (R7). |
| **States** | Loading / empty / error / success, per view. |
| **Cost** | Queries per session, refresh cadence, expected scan size. |
| **Security** | New secrets? New external calls? New persisted data? |
| **Evidence** | Claim → source → verified. (R2) |
| **Open questions** | What you need from the user. |

Then **stop and present it.** Do not write implementation code before the user replies.

> Here's the spec for `<feature>`. Two decisions I need from you: `<A>` and `<B>`.
> The DQL is validated — it returned 43 records over the last 24h. Approve and I'll build it.

If the user says "just build it", that is a valid answer — record it in the spec as
*approved without review* and proceed. The gate is about consent, not ceremony.

---

## Phase 3 — Build

Implement the spec, nothing else. Discoveries that change the design go back to the spec
with a one-line note; they do not get silently absorbed into the code.

Every symbol you write is verified ([verification-protocol.md](verification-protocol.md)).
Every component is Strato ([strato-components.md](strato-components.md)).

Suggested layout (from a working Dynatrace App):

```
app.config.json          # id, version, environmentUrl (placeholder!), scopes, icon
specs/                   # one .md per feature
ui/
  app/
    App.tsx              # routes
    components/          # shared presentational components
    pages/               # one file per route
    hooks/               # one hook per data source (useProblems, useAckEvents…)
    types/               # shared domain types — single source of truth
    utils/               # pure helpers (timeframe parsing, formatting)
  assets/                # icons, logos
api/                     # app functions (server-side JS runtime)
```

- **One hook per data source.** The query lives in the hook, not in the page.
- **Types in `types/`, imported by both page and hook.** No duplicated shapes.
- **Pages compose; they don't fetch and transform inline.**

---

## Phase 4 — Validate

Nothing here is optional. Run them in this order — the cheap checks first.

```bash
npx tsc --noEmit          # types
npm run lint              # ESLint incl. security + no-secrets rules
npx dt-app build          # it actually builds
```

Plus:

- **DQL executed** against the tenant, real records shown to the user (R6).
- **Data shape asserted and printed** before it reaches a component (R7).
- **The app opened in a browser** and the screen actually looked at (R7, R8).
- **The review checklist** run: [code-review-checklist.md](code-review-checklist.md) (R9).
- **The monotony test** passed: [strato-dataviz.md](strato-dataviz.md) (R8).

Report results honestly. If lint has 3 warnings, say "3 warnings, here they are" — not
"all checks pass".

---

## Phase 5 — Run / Deploy ⛔ GATE

`dt-app dev` freely. **`dt-app deploy` never without an explicit yes**, and always naming
the target:

> Ready to deploy **Command Center v1.2.0** to **`https://abc12345.apps.dynatrace.com`**.
> This publishes it to every user of that tenant. Deploy?

Details and commands: [app-lifecycle.md](app-lifecycle.md).

---

## Phase 6 — Publish ⛔ GATE

The version and the docs are **part of the change**, not follow-up work (R12):

1. Bump `app.config.json` → `app.version` per SemVer, and justify the level.
2. Add a dated `CHANGELOG.md` entry, written for the app's user.
3. Update the README per the sync matrix — cost figures included if the query changed.
4. Update `docs/index.html` to match. Same commit, no exceptions.
5. Sanitize sweep + secret scan on the staged content (R1).
6. Show the user the sanitize diff, the file list, and **the version**.
7. Commit everything together — no AI co-authorship (R10).
8. Push to the repository agreed in Phase 0, then tag `v<version>`.

Details: [release-and-docs-sync.md](release-and-docs-sync.md) and
[git-and-publishing.md](git-and-publishing.md).

---

## Keeping context across sessions

The next session starts with none of this in memory. Two files fix that, and both are worth
the minute they cost:

- **`AGENTS.md`** at the repo root — project-specific standing rules: which packages, which
  tables, which conventions, which traps you already hit. Point it at this skill.
- **`specs/`** — the design record. When behaviour changes, update the spec in the same
  commit as the code. A spec that has drifted from the code is worse than no spec.
