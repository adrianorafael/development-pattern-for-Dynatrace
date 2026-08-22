# Data visualization — right chart, right shape, proven with real data

> Rules **R7** (verified data shape before rendering) and **R8** (no monotonous screens).

Two distinct failures live here, and they look identical from the outside — an empty chart:

- **Shape failure.** The component is right, the data is the wrong shape, nothing renders,
  nothing errors. This is the single most common Dynatrace-App bug.
- **Choice failure.** Everything renders, and the screen is five line charts that answer
  no question. Technically working, practically useless.

---

## The chart inventory

Verified in `@dynatrace/strato-components@3.11.3` → `charts/index.d.ts` (2026-08).
Re-verify in the project's `node_modules`:

```bash
grep -oE "^export \{ [A-Za-z]+ \}" node_modules/@dynatrace/strato-components/charts/index.d.ts
```

| Component | Shows | Use when |
| --- | --- | --- |
| `TimeseriesChart` | Values over time (`.Line`, `.Area`, `.Bar`, `.Band` slots) | Trend, seasonality, before/after |
| `Sparkline` | Micro-trend, no axes | Trend inside a table cell or tile |
| `XYChart` | Generic X/Y with series slots | Non-time numeric relationships |
| `CategoricalBarChart` | Value per category (grouped / stacked) | Comparing discrete categories |
| `TopList` | Ranked bars, biggest first | "Top N offenders" |
| `PieChart` / `DonutChart` | Parts of a whole | 2–5 slices summing to a meaningful 100% |
| `HistogramChart` | Distribution of a numeric variable | Latency spread, "is this bimodal?" |
| `HoneycombChart` | One tile per entity, coloured by value | Fleet health at a glance, 20–500 entities |
| `TreeMap` | Hierarchical part-of-whole, area = size | Cost/usage breakdown with nesting |
| `SingleValue` / `SingleValueGrid` | One KPI, optional trend/threshold | Headline number |
| `GaugeChart` | One value against a range | SLO attainment, utilization vs. capacity |
| `MeterBarChart` / `MultiMeterBarChart` | Value against thresholds, as a bar | Compact capacity/health indicator |
| `AnnotationsChart` | Events on a timeline | Deploys, incidents alongside metrics |
| `DataTable` (`/tables`) | Exact values, many dimensions | The user needs to read, sort or export values |

Supporting parts: `ChartToolbar`, `ChartTooltip`, `ChartInteractions` (zoom/pan on
timeseries), `ChartSeriesAction`, `TimeseriesAnnotations`, `XYChartAnnotations`,
`*ChartConfig` context providers.

**Underscore-prefixed exports (`_SankeyChart`, `_FlameChart`, `_FlameGraph`,
`_MeterBarChartThresholdLegend`) are internal and unstable.** Do not use them in app code;
they can change or vanish without a major version.

`@dynatrace/strato-geo` covers maps. Gantt-style types exist in the package — confirm the
component itself is exported before you plan around it.

---

## R8 — Choose by question and shape, not by habit

Ask, in order: **What question does this answer? What shape is the data?**

| The question | Data shape | Reach for |
| --- | --- | --- |
| "How is it changing?" | value × time | `TimeseriesChart` |
| "Which is biggest?" | value × category, unordered | `CategoricalBarChart` |
| "Who are the worst offenders?" | value × category, ranked | `TopList` |
| "How is the whole divided?" | 2–5 parts of a total | `DonutChart` / `PieChart` |
| "How is the whole divided, with nesting?" | hierarchy + size | `TreeMap` |
| "How is it distributed?" | many samples of one number | `HistogramChart` |
| "Which of my 200 hosts is unhealthy?" | value per entity, many entities | `HoneycombChart` |
| "What is the number right now?" | one scalar | `SingleValue` (+ `SingleValueGrid` for a KPI row) |
| "Are we within budget/limit?" | value vs. threshold | `GaugeChart` / `MeterBarChart` |
| "What exactly happened, row by row?" | records with many fields | `DataTable` |
| "Did the deploy cause it?" | metric + discrete events | `TimeseriesChart` + `.Annotations` |

### The monotony test — run it before you call a screen done

For each screen, count the distinct visualization types. Then check:

- [ ] Not every panel is a line chart.
- [ ] There is a **headline layer** — `SingleValue`/`SingleValueGrid` answering "is it OK
      right now?" before the reader has to interpret a chart.
- [ ] Categorical comparisons use bars, not lines. *A line between unordered categories
      implies a continuum that does not exist — it is a factual error, not a style choice.*
- [ ] Rankings use `TopList`, not an alphabetical table.
- [ ] Fleet-scale entity health uses `HoneycombChart`, not 200 sparklines.
- [ ] At most one pie/donut per screen, with ≤5 slices.
- [ ] Where the user needs exact values, a `DataTable` exists — a chart is not a substitute.

**But variety is never the goal — fit is.** Three line charts showing three genuinely
different time series is correct. A honeycomb of four entities is not "more interesting",
it is worse than four `SingleValue` tiles. If you swap a chart type, be able to say which
question the new one answers better.

### When *not* to chart at all

- **1 value** → `SingleValue`.
- **2–3 values** → a `SingleValueGrid` or a small table usually beats any chart.
- **Precise values matter** (costs, IDs, timestamps) → table.
- **>5 pie slices** → `TopList` or `CategoricalBarChart` with an "Other" bucket.
- **The user will export it** → table, with `DataTableDownload`.

---

## R7 — Data contracts, and how to prove you satisfied one

### Contract A — the unified `ChartData` format

Most charts (honeycomb, categorical bar, pie/donut, treemap, top list, single value) take
the **unified flat-array format**:

```ts
// node_modules/@dynatrace/strato-components/charts/core/types/chart-data.d.ts
type ChartDatapoint = Record<string, unknown>;
type ChartData = ChartDatapoint[];
```

Fields are selected by **accessor strings** on the component, which support dot notation:

```tsx
<HoneycombChart
  data={[{ name: "host-a", value: 91 }, { name: "host-b", value: 42 }]}
  valueAccessor="value"      // defaults to "value"; supports "metrics.responseTime"
/>
```

Two things to get right, and both are silent when wrong:

1. **The accessor must name a key that exists on every object.** A typo yields blank tiles.
2. **Numeric fields must be numbers.** Grail returns some fields as strings; `"91"` is not
   `91` and will not colour a numeric hive.

### Contract B — the `Timeseries` format

`TimeseriesChart` takes an array of `Timeseries`:

```ts
// charts/core/types/timeseries.d.ts
interface TimeseriesDatapoint { start: Date; end?: Date; center?: Date; value: number }
interface Timeseries {
  datapoints: TimeseriesDatapoint[];
  name: string | string[];          // array = multi-dimensional series
  unit?: string;                    // e.g. "byte", "millisecond" — drives axis formatting
}
```

`start` is a **`Date` object**, not an ISO string and not epoch millis. This is the single
most common silent failure in Dynatrace App charts.

**Do not hand-roll this conversion.** Strato ships converters:

```tsx
import { convertQueryResultToTimeseries } from "@dynatrace/strato-components/charts";
import { useDql } from "@dynatrace-sdk/react-hooks";

const { data, isLoading, error } = useDql(query);
const series = useMemo(
  () => (data ? convertQueryResultToTimeseries(data) : []),
  [data],
);
```

Also exported: `convertToTimeseries(records, fieldTypes, hiddenFieldIds?, metadata?)`,
`convertToTimeseriesBand`, and `checkTimeseriesFormat`. Both converters **throw on
malformed data** — catch and surface it as an error state rather than letting it blank the
page.

The DQL that feeds them must produce timeseries records — `timeseries` or `makeTimeseries`,
not a bare `fetch`. → [dql-and-mcp.md](dql-and-mcp.md)

### Always set the unit

`unit: "millisecond"` turns `1500` into `1.5 s` on the axis and in the tooltip.
Without it the reader sees a bare number and has to guess. `@dynatrace-sdk/units` handles
formatting outside charts.

---

## The mandatory shape check

Before declaring a visualization done, do all four. Never skip 2.

**1. Verify the component's expected shape** — read its `.d.ts`, don't recall it:

```bash
find node_modules/@dynatrace/strato-components/charts/honeycomb -name '*.d.ts'
grep -rn "Accessor" node_modules/@dynatrace/strato-components/charts/honeycomb/types/*.d.ts
```

**2. Print the real data you are about to pass**, from the live query — not a mock:

```ts
console.log("chart input", JSON.stringify(chartData.slice(0, 3), null, 2));
console.log("types", Object.entries(chartData[0] ?? {}).map(([k, v]) => [k, typeof v]));
```

Read the output. Confirm: keys match the accessors; numbers are `number`; dates are `Date`.

**3. Assert the contract in code**, so the failure is loud instead of blank:

```ts
function assertChartData(rows: unknown, valueKey: string): asserts rows is ChartData {
  if (!Array.isArray(rows)) throw new Error("chart data must be an array");
  if (rows.length === 0) return;                      // empty is a valid state — render EmptyState
  const bad = rows.find((r) => typeof (r as never)[valueKey] !== "number");
  if (bad) throw new Error(`"${valueKey}" is not numeric: ${JSON.stringify(bad)}`);
}
```

**4. Render it and look at it.** Run `npx dt-app dev`, open the app, and confirm the chart
is populated, the axis units are right, the legend is readable, and the tooltip shows
sensible values. *A chart you have not looked at is not done.*

Then remove the debug logging — see [code-review-checklist.md](code-review-checklist.md).

---

## Every chart needs its four states

The same rule as any data view, and charts make it easy to forget:

```tsx
if (error)     return <MessageContainer variant="critical">{error.message}</MessageContainer>;
if (isLoading) return <ProgressCircle />;
if (!series.length) return <EmptyState>No data in the selected timeframe.</EmptyState>;
return <TimeseriesChart data={series} />;
```

Several charts also expose their own `.EmptyState` / `.ErrorState` slots — prefer those
where they exist, so the chart frame and legend stay in place.

---

## Colour

- Let Strato assign series colours. Its palettes are theme-aware and colour-blind conscious.
- Use `colorPalette` / `ColorRule` / threshold slots for **semantic** colouring — red means
  bad, not "the third series".
- Never a hardcoded hex. It will be wrong in one of the two themes.
- Keep a status colour consistent across every panel on a screen: if OPEN is red in the
  table, it is red in the chart.
