# Verification protocol — the cure for hallucination

> Rule **R2**. You may not name an API you have not verified in this session.

A hallucinated prop is not a small error. It compiles in JSX, renders nothing, throws no
warning, and costs an hour of debugging a component that was never wrong. The fix is
mechanical: **look it up, every time, before you type it.**

---

## The rule, stated precisely

Before emitting any of these, you must have read a source **in this session** that proves it:

- a Strato component name or subpath
- a component prop, slot, or sub-component (`TimeseriesChart.Legend`)
- an icon name
- a design token path
- a hook from `@dynatrace-sdk/react-hooks`
- a client method from `@dynatrace-sdk/client-*`
- a DQL command, function, or field name
- an OAuth scope string
- a `dt-app` command or flag

"I have used this before" is not a source. "It follows the naming convention" is not a
source. A `.d.ts` line or a doc page is a source.

---

## Source hierarchy

Use the highest available tier. Never skip up.

| Tier | Source | When |
| --- | --- | --- |
| **1** | `node_modules/**/*.d.ts` in the project | Always preferred for anything in an installed package. It is the exact version this app compiles against. |
| **2** | `developer.dynatrace.com` (fetched this session) | Semantics, guidelines, usage patterns, anything not expressible in types. |
| **3** | Official Dynatrace AI skills (`dynatrace/dynatrace-for-ai`, installed locally) | DQL syntax, Grail semantics, entity models, platform costs. |
| **4** | The npm tarball, when the package is not installed yet | `npm view`, or download and inspect. Records the exact version you inspected. |
| **5** | A live probe (MCP `dql_execute`, `npx dt-app help`) | The ultimate authority — it either runs or it doesn't. |
| ✗ | Memory, blog posts, StackOverflow, another repo's `AGENTS.md` | Hypothesis generators only. Confirm at tier 1–3 before use. |

Tier 4 note: other repositories' `AGENTS.md` and `CLAUDE.md` files describe the versions
*they* pinned. They go stale. `strato-components-preview` was where charts lived; it is now
deprecated. Never trust a second-hand inventory.

---

## The commands

### Does this component exist, and where is it exported from?

```bash
grep -rn "\bDataTable\b" node_modules/@dynatrace/strato-components/*/index.d.ts
# → tables/index.d.ts:  export { DataTable } from './DataTable/DataTable.js';
```

The directory name is the import subpath: `@dynatrace/strato-components/tables`.

### What are the real props?

```bash
find node_modules/@dynatrace/strato-components/tables -name 'DataTable*.d.ts'
sed -n '1,120p' node_modules/@dynatrace/strato-components/tables/DataTable/types/*.d.ts
```

Read the `Props` interface. Note which props are optional (`?`) and what the union types
actually permit — a prop typed `'sm' | 'md'` will not accept `"large"`.

### List everything a subpath exports

```bash
cat node_modules/@dynatrace/strato-components/charts/index.d.ts | grep -oE "export \{[^}]*\}" | head -50
```

### Does this icon exist?

```bash
grep -c "Icon }" node_modules/@dynatrace/strato-icons/index.d.ts     # how many there are
grep -n "RefreshIcon\|ReloadIcon" node_modules/@dynatrace/strato-icons/index.d.ts
```

Icon names are **not** guessable. `RefreshAutoIcon` and `AutoRefreshIcon` are not
interchangeable — one of them does not exist. Always grep.

### Which design token subpaths exist?

```bash
node -e "console.log(Object.keys(require('@dynatrace/strato-design-tokens/package.json').exports))"
```

### Which hooks does the SDK provide?

```bash
grep -oE "export declare function use[A-Za-z]+" node_modules/@dynatrace-sdk/react-hooks/types/index.d.ts | sort -u
```

### The package isn't installed yet

```bash
npm view @dynatrace/strato-components version
npm view @dynatrace/strato-components exports --json | head -40
```

Record the version you inspected in the evidence log. A claim verified against 3.11 is not
a claim about 3.0.

### Is this DQL valid?

Do not reason about it. Execute it. → [dql-and-mcp.md](dql-and-mcp.md)

### Is this `dt-app` command real?

```bash
npx dt-app help
npx dt-app deploy --help
```

---

## The evidence log

Every spec (and every non-trivial answer) carries one. It is three columns and it is
cheap to maintain:

```markdown
## Evidence

| Claim | Source | Verified |
| --- | --- | --- |
| `HoneycombChart` is exported from `@dynatrace/strato-components/charts` | `node_modules/@dynatrace/strato-components/charts/index.d.ts:112` | ✅ |
| Charts accept the unified `ChartData` = `Record<string, unknown>[]` format | `charts/core/types/chart-data.d.ts` | ✅ |
| Apps cannot configure `connect-src` CSP | https://developer.dynatrace.com/develop/security/configure-csp-rules/ | ✅ |
| `dt.davis.problems` exposes `event.status` | MCP `dql_execute` — 12 records returned | ✅ |
| Honeycomb tile colouring supports thresholds | — | ❓ unverified, not used |
```

A row marked ❓ may not appear in shipped code. Either verify it or design around it.

---

## Saying "I don't know"

When verification is impossible — package not installed, docs unreachable, no MCP — the
correct output is a question, not a guess:

> I can't verify `HoneycombChart`'s threshold prop: `@dynatrace/strato-components` isn't
> installed and `developer.dynatrace.com` is unreachable from here. Two ways forward:
> run `npm install` and I'll read the types, or paste the component's doc page. Meanwhile
> I've written the honeycomb with only the props I confirmed: `data`, `dataMappings`.

This is a **good** answer. It is short, it names the blocker, it offers two unblocks, and
it delivers the verified subset. Silence and a plausible-looking prop is a bad answer.

---

## Red flags in your own output

Stop and verify if you catch yourself writing any of these:

| Phrase | What it actually means |
| --- | --- |
| "should work" | not verified |
| "something like" | not verified |
| "typically the prop is called…" | not verified |
| "based on the standard React pattern" | you are about to import a non-Strato idiom |
| "I'll use `any` here to move on" | you did not read the type |
| "the docs probably say" | you did not read the docs |
| "as of my knowledge" | your knowledge is a hypothesis, the `.d.ts` is the answer |
