# Code review — read AI-generated code as if a stranger wrote it

> Rule **R9**. Run this before every push. Not before *some* pushes.

AI-generated code has a characteristic failure profile: it is syntactically clean,
idiomatic-looking, and wrong in ways that don't announce themselves. It optimises for
"looks like working code", and an inexperienced reviewer reads fluency as correctness.

Review your own output as adversarially as you would review a pull request from a stranger.

---

## Automated first — they're free

```bash
npx tsc --noEmit          # no implicit any, no missing props
npm run lint              # ESLint: security, SDL, no-secrets, no-unsanitized, react-hooks
npx dt-app build          # it compiles for production
```

The recommended ESLint stack — copy from
[`../assets/templates/eslint.config.mjs.template`](../assets/templates/eslint.config.mjs.template):

| Plugin | Catches |
| --- | --- |
| `typescript-eslint` (`recommendedTypeChecked`) | Unsafe `any` flows, floating promises |
| `eslint-plugin-security` | Common insecure patterns |
| `@microsoft/eslint-plugin-sdl` | `innerHTML`, `document.write` |
| `eslint-plugin-no-unsanitized` | Unsafe DOM sinks |
| `eslint-plugin-no-secrets` | High-entropy strings + a Dynatrace token regex |
| `eslint-plugin-react-hooks` | Rules of hooks, dependency arrays |
| `no-restricted-imports` | Strato root imports (R3) |

Keep `@typescript-eslint/no-deprecated` at `"error"` — it is what surfaces Strato
deprecations before a major bump breaks the build.

**Warnings are findings.** "Passes with 12 warnings" is not passing.

---

## 1. Secrets & data exposure (R1)

- [ ] No tokens, tenant URLs, tenant IDs, emails, or customer names in any staged file.
- [ ] `app.config.json` → `environmentUrl` is the placeholder.
- [ ] `.env` is ignored; `.env.example` is committed with placeholder values.
- [ ] No secret in a comment, a test fixture, or a commented-out block.
- [ ] `console.log` of query results, user objects or tokens removed. **Browser console
      output is visible to anyone who opens devtools.**
- [ ] Error messages surfaced to the user don't leak internal hostnames or raw stack traces.
- [ ] No real entity IDs or problem summaries in fixtures.

## 2. Injection & untrusted input

- [ ] **DQL is not built by string-concatenating user input.** Parameters or an allow-list.
- [ ] No `dangerouslySetInnerHTML`; no `innerHTML`, `outerHTML`, `document.write`.
- [ ] No `eval`, `new Function`, or `setTimeout("string")`.
- [ ] URLs built from data are validated before use — no `javascript:` schemes reaching an
      `href`; deep links into Dynatrace are properly encoded.
- [ ] Values rendered from Grail are treated as untrusted text, not as markup.

## 3. Permissions & platform boundaries

- [ ] Every scope in `app.config.json` is used by shipped code; unused ones deleted.
- [ ] Each scope carries a `comment` naming the feature that needs it.
- [ ] No write scope added to enable a read-only feature.
- [ ] No `fetch()` to an external domain from UI code — it is CSP-blocked; route it through
      an app function.
- [ ] Third-party credentials come from the Credential Vault inside an app function, never
      from the bundle.

## 4. Correctness

- [ ] **Loading, empty, error and success states all exist** for every data view.
- [ ] Async errors are caught and *shown*, not swallowed into a `catch {}`.
- [ ] No unhandled promise rejections (`@typescript-eslint/no-floating-promises`).
- [ ] Effects clean up: intervals cleared, subscriptions unsubscribed, fetches cancelled
      on unmount. **A leaked `setInterval` polling Grail is a leaked bill.**
- [ ] `useEffect` dependency arrays are complete and honest — no lint suppression to
      silence a real dependency.
- [ ] Optional/nullable Grail fields are handled; no `data.records[0].field` without a guard.
- [ ] Numbers from Grail that must be numeric are converted, not assumed
      ([strato-dataviz.md](strato-dataviz.md)).
- [ ] Timestamps are `Date` objects where the component requires `Date`.
- [ ] No `any` that hides a shape you never checked. `unknown` + a narrowing guard instead.

## 5. Strato & platform conventions (R3)

- [ ] No third-party UI or chart library.
- [ ] All Strato imports use subpaths, never the package root.
- [ ] No hardcoded hex/rgb colours — design tokens only, so both themes work.
- [ ] No hardcoded pixel margins where a `Flex`/`Grid` `gap` belongs.
- [ ] Interactive elements are real Strato controls, not `<div onClick>`.
- [ ] Icons verified to exist by name in `@dynatrace/strato-icons`.
- [ ] The app is wrapped in `AppRoot`.
- [ ] Navigation between Dynatrace apps uses `AppLink` / intents, not raw anchors.

## 6. Visualization (R7, R8)

- [ ] Each component's data contract verified against its `.d.ts`.
- [ ] Real data printed and inspected before it was wired up.
- [ ] Chart type chosen by data shape and question, not by habit.
- [ ] The monotony test passed.
- [ ] Units set on charts so axes read as `1.5 s`, not `1500`.
- [ ] The screen was opened in a browser and looked at.

## 7. Cost (DPS)

- [ ] Queries filter early and project only needed fields.
- [ ] Default timeframe is narrow.
- [ ] Auto-refresh is off by default, or at a defensibly slow interval.
- [ ] `scannedBytes` for the main query is known.
- [ ] No query inside a render path that re-fires every render.
- [ ] Cost implications documented in the README ([readme-and-page.md](readme-and-page.md)).

## 8. Maintainability

- [ ] No dead code, no commented-out experiments, no leftover `TODO: fix this`.
- [ ] No duplicated type definitions — shared types live in `types/`.
- [ ] Query logic lives in a hook, not inline in a page component.
- [ ] Names describe the domain (`acknowledgedProblems`), not the mechanism (`data2`).
- [ ] Comment density matches the surrounding file — don't narrate obvious lines.
- [ ] The spec was updated if behaviour changed.

## 9. Attribution (R10)

- [ ] No AI co-author trailer, no "Generated with", no session link.
- [ ] Commit author is the user's identity.

---

## Reporting the review

Be specific and honest. Findings the user can act on, ranked by severity:

> **Review — 2 findings**
>
> 🔴 `ui/app/hooks/useProblems.ts:34` — the timeframe string is interpolated straight into
> the DQL. If it ever comes from user input this is injection. Suggested: pass it as a
> query parameter.
>
> 🟡 `ui/app/pages/Problems.tsx:88` — `setInterval` isn't cleared on unmount. On a route
> change the poll keeps running: an invisible Grail query every 60s, billed.
>
> Everything else clean: `tsc` ✅, `eslint` ✅ (0 warnings), `dt-app build` ✅.

**Never report a check as passed if you did not run it.** If `dt-app build` was skipped
because dependencies are missing, say that — do not infer success from `tsc` passing.
