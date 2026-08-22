# Documentation research — depth-first, inside the Dynatrace world

> Rules **R4** (research before spec) and **R3** (stay inside `developer.dynatrace.com`).

Two failure modes this prevents:

1. **Shallow research.** Reading one overview page, then writing a spec on top of an
   overview's worth of understanding. Overviews exist to route you, not to inform you.
2. **Wandering off-platform.** Solving a Dynatrace problem with a Stack Overflow React
   answer, and shipping an app that does not look, behave, or perform like a native one.

---

## The canonical source map

`developer.dynatrace.com` is the only design and API authority for Dynatrace Apps, with
`docs.dynatrace.com` covering platform and product behaviour behind it.

**Fetch these at run time.** They are live URLs to be opened in the developer's own
environment during the session — not a summary to quote from memory. If a URL 404s,
**navigate down from the section index rather than guessing a new path**, and record the
URL that actually worked in your evidence log.

### Design system — Strato

| Topic | Entry point |
| --- | --- |
| **Design system home** | https://developer.dynatrace.com/design/ |
| About Strato | https://developer.dynatrace.com/design/about-strato-design-system/ |
| **Component catalogue** | https://developer.dynatrace.com/design/components/ |
| Components preview (legacy, deprecated) | https://developer.dynatrace.com/design/components-preview/ |
| **Data visualizations** | https://developer.dynatrace.com/design/data-visualizations/ |
| Data visualization basics | https://developer.dynatrace.com/design/foundations/data-visualization-basics/ |
| **Icons** | https://developer.dynatrace.com/design/icons/ |
| **Patterns** | https://developer.dynatrace.com/design/patterns/ |
| Pattern — app structure | https://developer.dynatrace.com/design/patterns/app-structure/ |
| Pattern — loading & saving | https://developer.dynatrace.com/design/patterns/loading-saving/ |
| Foundation — layout | https://developer.dynatrace.com/design/foundations/layout/ |
| Foundation — navigation | https://developer.dynatrace.com/design/foundations/navigation/ |
| Layout | https://developer.dynatrace.com/design/layout/ |
| Strato versioning | https://developer.dynatrace.com/design/strato-versioning/ |

Read **loading & saving** before building any view that fetches or persists — it is the
platform's answer to the four states (loading, empty, error, success) and to optimistic
saves, and it is the pattern most often reinvented badly.

Read **foundations/layout** and **foundations/navigation** before laying out a page or
adding a route. They define the spacing scale and the navigation hierarchy that make an app
feel native; guessing them is why an app "looks almost right" and nobody can say why.

### Platform & AppEngine

| Topic | Entry point |
| --- | --- |
| Introduction to Dynatrace Apps | https://developer.dynatrace.com/introduction/ |
| **About AppEngine** | https://developer.dynatrace.com/plan/about-appengine/ |
| AppEngine platform service | https://developer.dynatrace.com/plan/platform-services/app-engine/ |
| Get started / quickstart | https://developer.dynatrace.com/quickstart/ |
| Improve visualizations tutorial | https://developer.dynatrace.com/quickstart/tutorial/improve-visualizations/ |

### Develop

| Topic | Entry point |
| --- | --- |
| **Guides index** | https://developer.dynatrace.com/develop/guides/ |
| **Security guides** | https://developer.dynatrace.com/develop/guides/security/ |
| **Code optimization** | https://developer.dynatrace.com/develop/guides/code-optimization/ |
| Lazy loading | https://developer.dynatrace.com/develop/guides/code-optimization/lazy-loading/ |
| Configure CSP rules | https://developer.dynatrace.com/develop/security/configure-csp-rules/ |
| Custom CSP exceptions | https://developer.dynatrace.com/develop/security/custom-csp-exceptions/ |
| Secrets management | https://developer.dynatrace.com/develop/security/secrets-management/ |
| Query & visualize Grail data | https://developer.dynatrace.com/develop/data/query-and-visualize/ |
| Store app & user state | https://developer.dynatrace.com/develop/data/store-app-user-state/ |
| Intents | https://developer.dynatrace.com/develop/intents/about-intents/ |
| Workflows & custom actions | https://developer.dynatrace.com/develop/workflows/ |
| Navigation SDK | https://developer.dynatrace.com/develop/sdks/navigation/ |
| UI component shortcuts | https://developer.dynatrace.com/develop/ui-components/ |

### Release notes — check these when something behaves unexpectedly

| Topic | Entry point |
| --- | --- |
| App Toolkit | https://developer.dynatrace.com/release-notes/app-toolkit/ |
| Strato components | https://developer.dynatrace.com/release-notes/design-system/components-changelog/ |
| Strato components preview | https://developer.dynatrace.com/release-notes/design-system/components-preview-changelog/ |

### Dynatrace Docs — platform & product behaviour

`developer.dynatrace.com` tells you how to build. **https://docs.dynatrace.com** tells you
how the platform behaves, what it costs, and what the product already does.

| Topic | Entry point |
| --- | --- |
| Docs home | https://docs.dynatrace.com |
| AppEngine | https://docs.dynatrace.com/docs/platform/appengine |
| AppEngine Functions (DPS licensing) | https://docs.dynatrace.com/docs/license/capabilities/appengine-functions |
| Dynatrace MCP server | https://docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server |
| Credential vault | https://docs.dynatrace.com/docs/manage/credential-vault |
| DPS / platform subscription | https://docs.dynatrace.com/docs/manage/dynatrace-platform-subscription |

Go here for: DPS rate mechanics, Grail table availability, IAM and permission behaviour,
what a product feature already does before you rebuild it in an app.

### Adjacent, and still in-bounds

- **Official Dynatrace AI skills** — https://github.com/Dynatrace/dynatrace-for-ai for DQL
  and Grail semantics. **Fetch on demand, never vendor a copy** —
  → [dql-and-mcp.md](dql-and-mcp.md).
- **The installed packages** — `node_modules/@dynatrace*/**/*.d.ts` is the API's ground truth.
- **The house standard's reference implementation** —
  https://github.com/adrianorafael/Command-Center-for-Dynatrace and its
  [project page](https://adrianorafael.github.io/Command-Center-for-Dynatrace/) show the
  README and page structure this pattern expects.

---

## The depth protocol

**Never stop at the first page.** One page is the table of contents of your understanding.

```
1. LOCATE   Find the right section from the map above.
2. READ     Read that page end to end. Not the first paragraph.
3. EXPAND   Follow every in-page link that touches the feature:
              → the component's own page (props, use cases, do/don't)
              → linked foundations (layout, colour, typography, dataviz basics)
              → linked patterns (app structure, guided interaction, empty states)
              → linked guides (security, code optimization)
              → the changelog, when behaviour looks version-dependent
4. CROSS    Confirm the API against node_modules/**/*.d.ts.  ← the tiebreaker
5. LOG      Record URL + the one line it justified in the evidence log.
```

**Stop condition:** you can answer, without re-reading, (a) which component, (b) what data
shape it needs, (c) which props are required, (d) what the guidelines say *not* to do, and
(e) which scope or service it depends on. Any gap → keep expanding.

### How deep is deep enough — worked example

> Task: "show problem counts by severity"

| Depth | What you'd have | Verdict |
| --- | --- | --- |
| 0 | "I'll use a chart" | Useless |
| 1 | Read `/design/data-visualizations/` — charts exist | Still guessing |
| 2 | Read the specific chart page: it compares categories | Getting there |
| 3 | + read *data visualization basics*: which form fits categorical comparison; when a pie is wrong | Usable |
| 4 | + confirmed the component name and `data` shape in `charts/index.d.ts` and its `.d.ts` | **Ship it** |
| 5 | + ran the DQL through MCP and saw the real records | **Ship it with confidence** |

Depth 4 is the floor for writing code. Depth 5 is the floor for shipping a query.

---

## Staying in-platform

**Forbidden** as a source of solutions for a Dynatrace App:

- Third-party component libraries — MUI, Ant Design, Chakra, Bootstrap, shadcn/ui, Mantine
- Third-party chart libraries — Recharts, Chart.js, ECharts, Victory, Nivo, raw D3
- Utility CSS frameworks — Tailwind, Bulma
- Generic React dashboard tutorials, admin templates, "top 10 React chart libs" articles

**Why**, so you can explain it rather than just obey it:

1. **It stops looking native.** Users move between Dynatrace apps constantly; a foreign
   button, spacing scale or chart palette breaks that continuity instantly.
2. **Theming silently breaks.** Strato components follow the user's light/dark preference
   through design tokens. A hardcoded `#1496ff` is wrong in one theme, always.
3. **Accessibility regresses.** Strato ships keyboard navigation, focus management and ARIA
   that a hand-rolled `<div onClick>` does not.
4. **Bundle bloat.** A second chart library can double the bundle for one screen, and
   AppEngine app size is a real constraint.
5. **It breaks on upgrade.** Strato migrations (`dt-app update`, codemods) fix Strato code.
   Nothing migrates your bespoke component.

**When Strato genuinely lacks something**, the order is: (1) recheck the catalogue under a
different name — Strato's naming is not always the industry's; (2) compose it from Strato
primitives (`Flex`, `Grid`, `Surface`, `Container`) plus design tokens; (3) only then, and
only after telling the user it is a deviation, write bespoke code — styled with tokens,
never hex literals, and isolated in one file so it is easy to replace later.

Use `@dynatrace/strato-design-tokens/colors` (and `/spacings`, `/typography`, `/borders`)
for anything you style yourself. Never a raw hex.

---

## Recording what you learned

Research that lives only in the conversation is lost at the next session. Write findings
into the spec's evidence log ([spec-driven-workflow.md](spec-driven-workflow.md)), and when
a finding is durable — "charts want the unified `ChartData` flat-array format" — put it in
the project's `AGENTS.md` so the next session starts ahead instead of level.
