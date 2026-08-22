# Release & docs sync — the version and the docs move with the app

> Rule **R12**. Every publication carries a new version. Every push that changes what the
> app does, what it costs, or how it is set up updates the README and the project page in
> the same commit.

Two kinds of rot this prevents, and both are silent:

- **Version rot.** Three deploys all reporting `1.0.0`. Nobody — not the user, not the
  admin, not you six weeks later — can tell which build is running in which tenant.
- **Docs rot.** The README describes the app as it was two features ago. Someone installs
  it, follows the setup, and hits a scope error the README never mentions. The docs stop
  being read, because they stopped being true.

Both are cheap to prevent and expensive to discover.

---

## Part 1 — Versioning

### Where the version lives

`app.config.json` → `app.version` is the **single source of truth**. It is what Dynatrace
records at install time and what a user sees in the app listing.

```jsonc
"app": {
  "name": "Command Center",
  "version": "1.2.0",
  "id": "my.command.center"
}
```

Everything else — the git tag, the CHANGELOG heading, the README, the page — must agree
with it. A version that appears in two places with two values is worse than no version.

### Semantic versioning, in app terms

`MAJOR.MINOR.PATCH` — but "breaking" means *breaking for the person running the app*, not
for a library consumer. Translate it that way:

| Bump | When | Examples |
| --- | --- | --- |
| **MAJOR** `2.0.0` | The user must do something, or something they relied on is gone | A scope was added or changed (everyone must re-consent) · a persisted App State format changed without migration · a feature was removed · a default flipped in a way that changes cost · a required Grail table changed |
| **MINOR** `1.3.0` | New capability, nothing existing breaks | A new view, chart, filter or export · a new optional setting · a new read-only data source · a performance feature |
| **PATCH** `1.2.4` | Fix or polish, nothing new to learn | Bug fix · copy change · spacing/token fix · dependency bump with no behaviour change · a query optimized with identical results |

**Pre-1.0 (`0.x.y`) is the honest place for a demonstration app.** While the app is a study
version, stay on `0.x`: `0.MINOR.PATCH`, where MINOR absorbs breaking changes. Reaching
`1.0.0` is a statement that the shape is stable — do not make it by accident.

Pre-releases when you need to deploy something not yet ready: `1.3.0-beta.1`,
`1.3.0-rc.1`. Never reuse a pre-release number.

### Three rules that make the version trustworthy

1. **Never deploy the same version twice.** If a build reaches a tenant, its version is
   spent. Even for "just one more fix" — that is a PATCH bump.
2. **Bump in the same commit as the change**, not in a separate "bump version" commit.
   A commit that changes behaviour without touching the version is an incomplete commit.
3. **Tag what you publish.** `git tag -a v1.2.0 -m "…" && git push origin v1.2.0`.
   The tag is how someone gets back to the exact source of a build running in production.

### Deciding the bump — ask in this order

1. Does anyone have to re-consent, migrate, or change a setting? → **MAJOR**
2. Can a user do something they could not do before? → **MINOR**
3. Otherwise → **PATCH**

**The scope question is the one that gets missed.** Adding a scope to
`app.config.json` forces every existing user through a Dynatrace consent screen on next
load. That is a MAJOR change even when the code diff is three lines.

### CHANGELOG

Every published app carries a `CHANGELOG.md` in
[Keep a Changelog](https://keepachangelog.com/) form. Template:
[`../assets/templates/CHANGELOG.template.md`](../assets/templates/CHANGELOG.template.md).

```markdown
## [1.2.0] - 2026-08-22

### Added
- Assignee filter on the problems table.

### Changed
- Auto-refresh now defaults to **off**. It ran one Grail query per minute per open tab,
  which was the dominant DPS cost driver.

### Fixed
- Timestamps from Grail are converted to `Date` before reaching `TimeseriesChart`;
  the chart rendered empty for timeframes crossing midnight.
```

Write entries for the person running the app, not for the person who wrote the diff.
"Refactored `useProblems`" tells a user nothing. **Call out cost and scope changes
explicitly** — those are the two that cost someone money or an interrupted workflow.

---

## Part 2 — Docs sync

### The trigger

**Every time the user agrees to push, ask: does this change what the README or the page
says?** If yes, they change in the same commit as the code. Not "in a follow-up". A
follow-up that never comes is how the drift starts.

### What triggers what

| The change | README | Page | Version | CHANGELOG |
| --- | --- | --- | --- | --- |
| New feature or view | ✅ *App Features* | ✅ Features cards + stats | MINOR | ✅ |
| Feature removed | ✅ | ✅ | MAJOR | ✅ |
| New/changed OAuth scope | ✅ *Required OAuth Scopes* | ✅ scopes table | **MAJOR** | ✅ |
| New Grail table or query | ✅ *Cost* + *Behaviors* | ✅ Cost section + the query | MINOR | ✅ |
| Auto-refresh or timeframe default changed | ✅ **Cost** | ✅ **Cost** | MAJOR if cost rises | ✅ |
| Query optimized (fewer bytes) | ✅ Cost figures | ✅ Cost figures | PATCH | ✅ |
| New setup or config step | ✅ *Installation* | ✅ Quick start | MINOR | ✅ |
| New placeholder in a committed file | ✅ *Installation* | ✅ Quick start | PATCH | ✅ |
| New known limitation | ✅ *Important Behaviors* | ✅ How it works | — | ✅ |
| New error someone will hit | ✅ *Troubleshooting* | ✅ Troubleshooting | — | — |
| File structure changed | ✅ *File Structure* | ✅ Structure tree | — | — |
| Dependency bump, no behaviour change | — | — | PATCH | ✅ |
| Internal refactor, no behaviour change | — | — | PATCH | optional |
| Typo, spacing, colour token | — | — | PATCH | — |

A dash means *genuinely nothing to say*. It is not permission to skip the check.

### README and page are one document at two depths

They are not independent. The page is the shop window; the README is the manual. They must
agree on:

- the disclaimer (identical wording)
- the scope list
- the cost figures and the scenario table
- the troubleshooting rows
- the prerequisites
- the file structure
- **the version they describe**

When one changes, open the other. Divergence between them is the most common rot in this
pattern, and it is invisible until a reader notices the page promises a feature the README
has never heard of.

### The publish checklist

Run this every time, before the push. It is nine lines and it is not optional.

- [ ] `app.config.json` → `app.version` bumped, and the bump level justified above
- [ ] `CHANGELOG.md` has an entry, dated, written for the app's user
- [ ] README updated per the matrix — **including cost figures if the query changed**
- [ ] `docs/index.html` updated to match the README, same commit
- [ ] Version consistent across `app.config.json`, CHANGELOG, README, page
- [ ] Spec in `specs/` reflects the change ([spec-driven-workflow.md](spec-driven-workflow.md))
- [ ] Sanitize + secret scan clean ([security-and-secrets.md](security-and-secrets.md))
- [ ] Review checklist run ([code-review-checklist.md](code-review-checklist.md))
- [ ] Git tag prepared for the version being published

Then tell the user what you are about to push, naming the version:

> Ready to push **v1.2.0** — 3 commits. README and page both updated: new scope
> `document:documents:read` in the scopes table, cost section re-measured at ~3 MB/query
> (was ~5 MB). Secret scan clean. Push, and tag `v1.2.0`?

### Keeping the page deployable

If the repository publishes `docs/` through a `gh-pages` branch, that branch is
**generated** — never hand-edited. Edit `docs/`, let CI regenerate it.
→ [readme-and-page.md § Turning Pages on](readme-and-page.md)

---

## Part 3 — The release flow, end to end

```
1. Change is built and validated                    → phases 3-4
2. Decide the bump: MAJOR / MINOR / PATCH           → the three questions above
3. app.config.json → app.version                    → one source of truth
4. CHANGELOG.md → new dated section                 → written for the app's user
5. README.md    → per the sync matrix               → cost figures re-measured if the query changed
6. docs/index.html → mirror the README              → same commit, no exceptions
7. Sanitize + scan + review                         → R1, R9
8. Commit (code + version + docs together)          → no AI attribution, R10
9. ⛔ Ask, naming the version and the target        → then push
10. Tag v<version> and push the tag
11. ⛔ Ask, naming the tenant                       → then npx dt-app deploy
```

Steps 3–6 belong to the same commit as step 1. Splitting them is how a repository ends up
with a version that means nothing and a README nobody trusts.
