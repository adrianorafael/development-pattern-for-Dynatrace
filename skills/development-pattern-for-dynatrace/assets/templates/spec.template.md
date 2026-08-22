# Spec — <Feature name>

**Status:** draft | ⛔ awaiting approval | approved | implemented
**Date:** YYYY-MM-DD
**App:** `<App Name> for Dynatrace`

---

## Goal

<One sentence. What can the user do after this that they cannot do now?>

## Non-goals

- <Explicitly out of scope, so it does not creep in later>
- <…>

## Data

**Grail table(s):** `<dt.davis.problems | logs | spans | metrics | …>`

**DQL:**

```dql
fetch dt.davis.problems, from: $from, to: $to
| filter isNull(event.end)
| fields display_id, event.status, event.start, event.name
| sort event.start desc
| limit 500
```

| Check | Result |
| --- | --- |
| Executed against a live tenant | ✅ / ❌ |
| Records returned | `<n>` over `<timeframe>` |
| Field names & types confirmed | ✅ / ❌ |
| User confirmed this is the right data | ✅ / ❌ |
| `scannedBytes` per run | `<n>` |

Sample record (real, from the validation run — **no tenant-identifying values**):

```json
{ "display_id": "P-00001", "event.status": "ACTIVE", "event.start": "2026-08-01T10:00:00Z" }
```

> If any check above is ❌, this spec is not ready. → `references/dql-and-mcp.md`

## Scopes

| Scope | Justification |
| --- | --- |
| `storage:events:read` | DQL access to `dt.davis.problems` |
| | |

New scopes force every existing user to re-consent on next load.

## UI

**Route / page:** `<path>`

| Component | Subpath |
| --- | --- |
| `Page`, `Flex` | `@dynatrace/strato-components/layouts` |
| `DataTable` | `@dynatrace/strato-components/tables` |
| | |

Layout sketch:

```
┌──────────────────────────────────────────┐
│ TitleBar + TimeframeSelector             │
├──────────────────────────────────────────┤
│ SingleValueGrid — 4 KPIs                 │
├──────────────────┬───────────────────────┤
│ CategoricalBar   │ HoneycombChart        │
├──────────────────┴───────────────────────┤
│ DataTable                                │
└──────────────────────────────────────────┘
```

## Visualization

| Panel | Component | Why this one for this data |
| --- | --- | --- |
| Status breakdown | `CategoricalBarChart` | Unordered categories — a line would imply a false continuum |
| Host health | `HoneycombChart` | ~200 entities, one value each; a per-entity chart would be unreadable |
| | | |

**Monotony test:** distinct visualization types on this screen: `<n>`. Headline layer
present: ✅ / ❌. → `references/strato-dataviz.md`

## Data contract

| Component | Expected shape | Verified in |
| --- | --- | --- |
| `HoneycombChart` | `ChartData` = `Record<string, unknown>[]`, `valueAccessor="value"`, value must be `number` | `charts/core/types/chart-data.d.ts` |
| `TimeseriesChart` | `Timeseries[]` — `datapoints[].start` is a `Date` | `charts/core/types/timeseries.d.ts` |

Conversion: `<convertQueryResultToTimeseries(data)> / <custom mapper in hooks/useX.ts>`

## States

| View | Loading | Empty | Error |
| --- | --- | --- | --- |
| `<panel>` | `ProgressCircle` | `EmptyState` — "<copy>" | `MessageContainer variant="critical"` |

## Cost (DPS)

| | |
| --- | --- |
| Queries per app open | `<n>` |
| Auto-refresh | off / `<interval>` |
| Default timeframe | `<e.g. Today>` |
| Scanned bytes per run | `<n>` |
| Estimated GB/month at `<n>` users | `<n>` |

Mitigations applied: `<projected fields / narrow default / refresh off by default>`

## Security

- New secrets? `<none / which, and where they live>`
- External calls? `<none / via app function X>`
- Data persisted? `<none / App State key, shared across tenant users, 400KB cap>`
- User input reaching DQL? `<none / parameterized>`

## Evidence

| Claim | Source | Verified |
| --- | --- | --- |
| | | ✅ |
| | | ❓ |

> ❓ rows may not appear in shipped code. Verify them or design around them.

## Open questions

1. <What you need from the user before building>
