# Strato components — the native UI, and only the native UI

> Rules **R3** (Strato only) and **R2** (verify before use).

Strato is the Dynatrace design system. An app built from Strato *is* a Dynatrace app.
An app built from anything else is a web page that happens to live inside Dynatrace.

---

## Packages

| Package | Contains | Status |
| --- | --- | --- |
| `@dynatrace/strato-components` | **Everything**: layouts, typography, buttons, forms, tables, charts, filters, navigation, overlays, content, editors | ✅ Use this |
| `@dynatrace/strato-design-tokens` | Colors, spacings, typography, borders, box-shadows, elevations, breakpoints, animations | ✅ Use this |
| `@dynatrace/strato-icons` | ~420 icon components | ✅ Use this |
| `@dynatrace/strato-geo` | Map / geo visualization primitives | ✅ When you need maps |
| `@dynatrace/strato-components-preview` | Thin re-export shim of `@dynatrace/strato-components` | ⚠️ **Deprecated** in 3.x |

### The preview package is deprecated — read this before copying an old example

Older tutorials, older `AGENTS.md` files, and most model training data say charts, tables
and filters live in `strato-components-preview`. **That was true; it no longer is.**
In Strato 3.x the preview package's own README says *"Strato components preview
(Deprecated) … Switch to strato-components"*, and its subpath indexes are one line each:

```ts
// node_modules/@dynatrace/strato-components-preview/charts/index.d.ts
export * from '@dynatrace/strato-components/charts';
```

New code imports from `@dynatrace/strato-components`. If the project still has the preview
package installed, existing imports keep working — migrate them opportunistically, don't
churn the diff for it.

---

## Import rules

**Always import from the subpath. Never from the package root.**

```ts
// ❌ breaks tree-shaking, blocked by the project's ESLint rule
import { Flex, Heading, DataTable } from "@dynatrace/strato-components";

// ✅
import { Flex, Grid, Surface } from "@dynatrace/strato-components/layouts";
import { Heading, Text, Paragraph } from "@dynatrace/strato-components/typography";
import { DataTable } from "@dynatrace/strato-components/tables";
import { TimeseriesChart } from "@dynatrace/strato-components/charts";
import Colors from "@dynatrace/strato-design-tokens/colors";
import { RefreshIcon } from "@dynatrace/strato-icons";
```

`@dynatrace/strato-icons` has a single root export — icons are the one flat import.

The ESLint rule that enforces this is in
[`../assets/templates/eslint.config.mjs.template`](../assets/templates/eslint.config.mjs.template).
Keep it at `"error"`.

---

## Subpath → component inventory

Verified against `@dynatrace/strato-components@3.11.3` (2026-08). **Re-verify in the
project's own `node_modules`** — this is a map, not a contract:

```bash
grep -oE "export \{[^}]*\}" node_modules/@dynatrace/strato-components/<subpath>/index.d.ts
```

| Subpath | Components |
| --- | --- |
| `/layouts` | `Page`, `PageLayout`, `AppHeader`, `AppNavLink`, `TitleBar`, `Container`, `Surface`, `Flex`, `Grid`, `Divider`, `InputGroup`, `HelpMenu`, `Logo` |
| `/typography` | `Heading`, `Text`, `Paragraph`, `Link`, `ExternalLink`, `List`, `Code`, `Blockquote`, `Strong`, `Emphasis`, `Highlight`, `Strikethrough`, `TextEllipsis` |
| `/buttons` | `Button`, `IntentButton`, `NotifyButton`, `RunQueryButton` |
| `/forms` | `TextInput`, `TextArea`, `NumberInput`, `PasswordInput`, `SearchInput`, `Select` (+ `SelectOption`, `SelectGroup`, `SelectTrigger`, `SelectFilter`, `SelectEmptyState`, …), `Checkbox`, `Radio`, `RadioGroup`, `Switch`, `ToggleButtonGroup`, `DateTimePicker`, `FormField`, `FieldSet`, `Label`, `Hint`, `FormFieldMessages` |
| `/tables` | `DataTable` (+ `DataTablePagination`, `DataTableToolbar`, `DataTableDownload`, `DataTableCellActions`, `DataTableExpandableRowTemplate`, `DataTableConfigProvider`), `SimpleTable`, `TableActionsMenu` |
| `/charts` | see [strato-dataviz.md](strato-dataviz.md) |
| `/filters` | `FilterBar`, `FilterField`, `TimeframeSelector`, `SegmentSelector`, `SegmentsProvider`, `TIMEFRAME_SELECTOR_PRESETS` |
| `/navigation` | `AppLink`, `Breadcrumbs`, `Menu`, `Tabs`, `Tab` |
| `/overlays` | `Modal`, `Sheet`, `Overlay`, `Tooltip`, `DismissButton` |
| `/content` | `EmptyState`, `MessageContainer`, `Chip`, `ChipGroup`, `Accordion`, `HealthIndicator`, `ProgressCircle`, `ProgressBar`, `Skeleton`, `SkeletonText`, `CodeSnippet`, `Markdown`, `Avatar`, `AvatarGroup`, `ExpandableText`, `InformationOverlay`, `TerminologyOverlay`, `Microguide`, `FeatureHighlight`, `KeyboardShortcut`, `ReleasePhase`, `AiResponse`, `AiLoadingIndicator` |
| `/editors` | `CodeEditor`, `DQLEditor` |
| `/core` | `AppRoot`, `Intent`, `OverlayProvider`, `ScrollProvider`, `TIMEFRAME_EXPRESSION`, permission types |
| `/notifications` | `ToastContainer`, `NotificationSettings` |

**`AppRoot` wraps the whole app.** Without it, theming and overlays do not work.

---

## Composition patterns that read as native

### Page shell

`Page` provides `Header`, `Sidebar`, `Main` and `DetailView` slots. `Main` is the only
mandatory one; the layout handles small screens itself, moving `Sidebar`/`DetailView` into
a drawer. Do not rebuild that with your own media queries.

```tsx
<AppRoot>
  <Page>
    <Page.Header><AppHeader>{/* nav + app-wide actions */}</AppHeader></Page.Header>
    <Page.Main>{/* content */}</Page.Main>
  </Page>
</AppRoot>
```

### The four states of every data view

A view that only handles the happy path is unfinished. All four, every time:

| State | Component |
| --- | --- |
| Loading | `ProgressCircle`, or `Skeleton`/`SkeletonText` for layout-preserving loads |
| Empty | `EmptyState` — with a sentence saying *why* it is empty and what to do |
| Error | `MessageContainer` with `variant="critical"` — show the actual error, not "Something went wrong" |
| Success | the table/chart |

### Layout and spacing

Use `Flex` and `Grid` with the `gap` prop and spacing tokens. Never a `margin: 12px`
literal, never a magic-number `<div style>`.

```tsx
import { Flex } from "@dynatrace/strato-components/layouts";
<Flex flexDirection="column" gap={16}>…</Flex>
```

### Styling anything bespoke

```ts
import Colors from "@dynatrace/strato-design-tokens/colors";
import Spacings from "@dynatrace/strato-design-tokens/spacings";
// Colors.Text.Primary.Default, Colors.Background.Surface.Default, …
```

Available token subpaths: `colors`, `spacings`, `typography`, `borders`, `box-shadows`,
`elevations`, `breakpoints`, `animations`, `easings`, `timings`, `variables`,
`variables-dark`. Confirm exact token names by reading the package's `.d.ts` — token names
are as unguessable as icon names.

**A raw hex colour in an app is a theming bug waiting for a user in dark mode.**

### Icons

```bash
# find one — never guess the name
grep -n "Refresh\|Reload" node_modules/@dynatrace/strato-icons/index.d.ts
```

```tsx
import { RefreshIcon } from "@dynatrace/strato-icons";
<Button><Button.Prefix><RefreshIcon /></Button.Prefix>Refresh</Button>
```

Icons carry meaning. Reuse the icon Dynatrace already uses for a concept (problems, hosts,
services, logs) rather than inventing a new visual vocabulary.

---

## `DataTable` essentials

`DataTable` is the right choice for anything sortable, filterable, paginated or selectable.
`SimpleTable` is for static tabular content (typically Markdown rendering).

- Required props: `data` (array of row objects) and `columns`.
- Each column needs `id`, `header`, and an `accessor` (a string path or a function).
- Use the built-in sorting/pagination/selection rather than hand-rolling them — you get
  keyboard support and screen-reader semantics for free.
- Read the current props before use:
  `find node_modules/@dynatrace/strato-components/tables -name '*.d.ts' | head`.

Row count matters: past a few hundred rows, paginate or aggregate in DQL rather than
shipping everything to the browser. → [dql-and-mcp.md](dql-and-mcp.md)

---

## Forbidden

| Never | Instead |
| --- | --- |
| MUI, Ant, Chakra, Bootstrap, shadcn/ui, Mantine | Strato components |
| Recharts, Chart.js, ECharts, Victory, Nivo, raw D3 | `@dynatrace/strato-components/charts` |
| Tailwind, Bulma, global CSS resets | `Flex`/`Grid` + design tokens |
| Hardcoded hex/rgb colours | `@dynatrace/strato-design-tokens/colors` |
| `<div onClick>` as a button | `Button` |
| `window.alert` / `confirm` | `Modal`, `MessageContainer`, `ToastContainer` |
| `dangerouslySetInnerHTML` | `Markdown`, or Strato typography |
| `<a href>` to another Dynatrace app | `AppLink` / the Navigation SDK / intents |
| A custom theme toggle | The platform's theme, consumed via tokens |

---

## Performance

- **Code-split heavy routes** with `React.lazy` + `Suspense`. The toolkit supports
  fine-grained async module loading; a chart-heavy page should not be in the initial bundle.
  → https://developer.dynatrace.com/develop/guides/code-optimization/lazy-loading/
- **Subpath imports** (above) are half of bundle hygiene.
- **Check what you shipped**: `npx dt-app analyze` for a bundle report.
- **Memoize expensive transforms.** Converting Grail records to chart data on every render
  is a common, invisible cost — do it in a `useMemo` keyed on the query result.
