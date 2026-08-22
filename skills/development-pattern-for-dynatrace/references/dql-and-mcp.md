# DQL & the Dynatrace MCP server — never ship an unexecuted query

> Rule **R6**. A DQL query that has not run against a real tenant is a hypothesis.

DQL is not SQL and it is not KQL. Its syntax differs in ways that *look* fine to a model
trained on other query languages — `filter x in ["a","b"]` is wrong, `filter in(x, {"a","b"})`
is right — and a wrong query rarely errors loudly. It returns nothing, or the wrong thing.

---

## Step 1 — Read the official Dynatrace knowledge, fresh, at the moment you need it

Dynatrace publishes its own agent skills at
**https://github.com/Dynatrace/dynatrace-for-ai** — plain Markdown, maintained by
Dynatrace, authoritative for DQL syntax, Grail semantics, entity models and platform costs.

**Reference them. Do not vendor them.**

This skill deliberately ships **no copy** of that content, and neither should your project.
Dynatrace updates those files — DQL gains functions, semantic dictionaries change, cost
guidance shifts — and a copy taken during one session is stale by the next. A stale copy is
worse than no copy, because it looks authoritative while being wrong.

So: **fetch the specific file you need, at the moment you need it, every time.**

```bash
# Read one skill directly from the source. Nothing persists.
curl -sSL https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/skills/dt-dql-essentials/SKILL.md

# ...then the reference it routes you to
curl -sSL https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/skills/dt-dql-essentials/references/semantic-dictionary.md
```

In an agent with web access, fetching the GitHub URL directly is equivalent and preferable.

Rules for handling them:

- **Never commit them** to the app repository — not as a vendored directory, not as a
  quoted excerpt presented as current truth.
- **Never cache them across sessions.** If you must write to disk, write to a temp path
  outside the repository and treat it as valid for this task only.
- **Re-fetch when the task changes.** Reading `dt-dql-essentials` an hour ago does not
  cover the `dt-obs-kubernetes` question you are on now.
- **Cite the commit or the date** when a claim in a spec's evidence log depends on one.

If the developer has separately installed them into their own agent
(`npx skills add dynatrace/dynatrace-for-ai`, or the Claude Code plugin), that is their
environment's business — use what is loaded, and still prefer a fresh fetch when a claim is
load-bearing. What must not happen is *this* skill or *your app repository* carrying a
frozen copy.

The ones that matter most here:

| Skill | Read it before |
| --- | --- |
| `dt-dql-essentials` | **any** DQL — syntax, pitfalls, semantic dictionary, optimization |
| `dt-obs-problems` | querying `dt.davis.problems`, root cause, impact |
| `dt-obs-logs` | log queries, patterns, correlation |
| `dt-obs-services` / `-hosts` / `-kubernetes` | entity-specific fields and metric keys |
| `dt-platform-costs` | actual DPS consumption and chargeback queries |
| `dt-js-runtime` | app functions: contract, limits, `@dynatrace-sdk/*` catalog |
| `dt-app-dashboards` / `dt-app-notebooks` | tiles, variables, notebook structure |
| `dt-sec-insights` | `security.events` |

Read the relevant `SKILL.md` **and** the `references/*.md` it routes you to. Its
`references/semantic-dictionary.md` is the answer to "what is this field actually called".

The path pattern is stable even though the catalogue is not:

```
https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/skills/<skill>/SKILL.md
https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/skills/<skill>/references/<file>.md
```

**The list above is a snapshot, not the catalogue.** Dynatrace adds and renames skills.
When you need one that is not listed — or want to confirm a name — read the repository's
own README first:
`https://raw.githubusercontent.com/Dynatrace/dynatrace-for-ai/main/README.md`.
That is the live index; this table is a shortcut that will drift.

---

## Step 2 — Decide whether you need the MCP server

| You are… | MCP needed? |
| --- | --- |
| Writing or changing any DQL | **Yes** — R6 requires execution |
| Checking which fields a table actually has | **Yes** — `fieldsSummary` on the real tenant |
| Estimating query cost | **Yes** — scanned bytes come from the real run |
| Only changing layout/styling with no query | No |

**Detect before asking.** List the tools available in the current session and look for a
Dynatrace server. If it is there, use it. If it is not, say so and offer setup — do not
quietly skip validation and do not pretend a query was verified.

If the user declines to connect it, that is their call. Then every query you write is
labelled explicitly:

> ⚠️ Unvalidated — this query has not been executed against a tenant. Run it in a Notebook
> and confirm the fields before relying on it.

---

## Step 3 — Connect it (securely)

The server is HTTP, served by the tenant itself:

```jsonc
// .mcp.json — committed; note the values are ${ENV} references, never literals
{
  "mcpServers": {
    "dynatrace": {
      "type": "http",
      "url": "${DT_ENVIRONMENT}/platform-reserved/mcp-gateway/v0.1/servers/dynatrace-mcp/mcp",
      "headers": { "Authorization": "Bearer ${DT_PLATFORM_TOKEN}" }
    }
  }
}
```

```bash
# .env — gitignored. Real values live ONLY here.
DT_ENVIRONMENT=https://YOUR-ENVIRONMENT.apps.dynatrace.com
DT_PLATFORM_TOKEN=dt0s16.YOUR-PLATFORM-TOKEN
```

🔒 **Three hard rules** (see [security-and-secrets.md](security-and-secrets.md)):

1. `.mcp.json` may contain `${VAR}` references. It may **never** contain a literal token or
   a literal tenant URL. Check this before every commit.
2. `.env` is gitignored from the start. Ship a committed `.env.example` beside it.
3. The platform token gets the **minimum** scopes for what you actually query. Read/query
   scopes only, unless the app genuinely writes. The Dynatrace MCP server docs
   (`docs.dynatrace.com/docs/shortlink/dynatrace-mcp-server`) list the required set.

Also usable: **dtctl**, the Dynatrace CLI (`dynatrace-oss/dtctl`), which ships its own
agent skill and authenticates per-context — a good fit when you prefer not to mint a
long-lived token.

---

## Step 4 — The validation loop

Never write a query top-to-bottom and run it once. Build it in layers, executing at each.

```
1. DISCOVER   What exists?
              fetch <table> | limit 1
              fetch <table> | fieldsSummary   ← the real field names and types
2. NARROW     Add filters. Execute. Row count should drop for the reason you expect.
3. SHAPE      Add summarize / makeTimeseries. Execute. Inspect the output columns.
4. CONFIRM    Show the user the real records and ask: "is this the data you meant?"
5. COST       Note scannedBytes / scannedRecords from the run metadata.
6. PIN        Paste the final query, verbatim, into the spec's evidence log.
```

**Never invent a field name.** `fieldsSummary` and the semantic dictionary in
`dt-dql-essentials` are the two legitimate sources.

### What "validated" means

A query is validated only when you can state all five:

- [ ] It executed without error against the target tenant.
- [ ] It returned records (or you confirmed *empty is correct* for this timeframe).
- [ ] The field names and types in the result match what the UI consumes.
- [ ] The user has seen a sample of the real output and confirmed it is the right data.
- [ ] The scanned-bytes figure is known and acceptable.

Anything less gets the ⚠️ Unvalidated label.

### Getting the same rigour without MCP

Fallbacks, in order of preference — all still require a human in the loop:

1. Paste the query into a **Dynatrace Notebook** and read the result + scan metadata.
2. Use **dtctl** from the terminal.
3. In-app, log `metadata.grail.scannedBytes` from `queryExecute` during `dt-app dev`.

---

## DQL syntax traps worth memorizing

From `dt-dql-essentials` — but read the full list there, this is only the top of it.

| ❌ Wrong | ✅ Right | Why |
| --- | --- | --- |
| `filter field in ["a","b"]` | `filter in(field, {"a","b"})` | `[…]` wraps sub-queries; static arrays use `{}` or `array()` |
| `by: severity, status` | `by: {severity, status}` | field lists in `by:` need braces |
| `contains(lower(f), "err")` | `contains(f, "err", false)` | third positional arg is `caseSensitive` |
| `fetch logs \| filter …` late | filter **immediately** after `fetch` | filtering early is what makes it cheap |
| `fetch logs` (all fields) | `fetch logs \| fields ts, content, …` | Grail is columnar — fewer fields, fewer bytes |
| Wide default timeframe | narrow it, or pass `from:`/`to:` | timeframe is the dominant cost lever |

---

## In the app: querying from React

Prefer the hook. It handles state, cancellation and errors:

```tsx
import { useDql } from "@dynatrace-sdk/react-hooks";

const { data, error, isLoading } = useDql(query);
```

Verified hooks in `@dynatrace-sdk/react-hooks` (confirm in the project's own copy —
`grep -oE "export declare function use[A-Za-z]+" node_modules/@dynatrace-sdk/react-hooks/types/index.d.ts`):
`useDql`, `useDqlQuery`, `useGrailFields`, `useDocument`, `useListDocuments`,
`useCreateDocument`, `useUpdateDocument`, `useAppState`, `useUserAppState`,
`useSetAppState`, `useSetUserAppState`, `useSettings`, `useAppFunction`, `useUser`,
`useUsers`, `useAnalyzer`, `useEffectivePermissions`.

Drop to `@dynatrace-sdk/client-query` (`queryClient.queryExecute`) only outside React or
when you need the raw metadata (scanned bytes) — then still surface errors in the UI.

**Rules for query code:**

- **Build queries from constants, never by concatenating user input.** A user-controlled
  string spliced into DQL is an injection. Pass values through query parameters, or
  validate against an allow-list you own.
- **Declare the matching scope** in `app.config.json` — `storage:events:read` for
  `dt.davis.problems`, `storage:logs:read` for logs, etc. Nothing more.
- **Rebuild the query only when its inputs change**, and memoize; a query string rebuilt
  every render re-fetches every render.
- **Never poll faster than the data changes.** Auto-refresh is the number-one DPS cost
  driver in real apps.

---

## Cost awareness — this is a billed operation

Grail queries consume DPS budget, charged on **data analyzed (GB scanned)**. Every
execution costs, and the multipliers compound:

```
GB/month ≈ GB_per_query × queries_per_hour × hours_per_day × days × concurrent_users
```

| Lever | Effect |
| --- | --- |
| Auto-refresh interval | 🔴 Dominant — one query per interval **per open tab** |
| Timeframe width | More data scanned per run |
| Concurrent users / open tabs | Multiplies linearly |
| Projected fields (`\| fields …`) | Fewer columns, fewer bytes |
| Filter placement | Filter early, scan less |

**Measure, don't estimate.** `queryExecute` returns `metadata.grail.scannedBytes` and
`scannedRecords` — the true per-run cost. Aggregate consumption is visible in
*Account Management → Cost & usage → Grail Query*.

Design defaults accordingly: narrow default timeframe, auto-refresh **off by default** or
at a slow interval, projected fields, and — per
[readme-and-page.md](readme-and-page.md) — a **cost section in the README and on the page**
so whoever installs the app is not surprised by their bill.
