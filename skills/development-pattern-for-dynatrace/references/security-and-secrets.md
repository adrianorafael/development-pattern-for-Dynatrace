# Security & secrets — nothing sensitive reaches the repository

> Rule **R1**. This is the one failure that cannot be undone by a follow-up commit.
> A pushed secret is a leaked secret: Git history, forks, GitHub's event API, and code
> search caches keep it. Rotation is the only remedy — prevention is the only strategy.

---

## 1. What counts as sensitive in a Dynatrace App

| Class | Examples | Why it matters |
| --- | --- | --- |
| **Platform tokens** | `dt0c01.ABC…`, `dt0s16.…`, OAuth client secrets, `DT_PLATFORM_TOKEN` | Direct API access to the tenant |
| **Tenant identity** | `abc12345`, `https://abc12345.apps.dynatrace.com`, `https://abc12345.live.dynatrace.com`, SaaS/Managed cluster hostnames | Identifies the customer; enables targeted attack |
| **Auth artifacts** | `.dt-app/` (holds cached OAuth tokens), `*.pem`, `*.key`, `*.p12`, `credentials.json` | Replayable credentials |
| **Environment files** | `.env`, `.env.local`, `.env.production` | Almost always contains both of the above |
| **People** | Real user emails, usernames, on-call phone numbers, escalation contacts | Personal data; often regulated |
| **Business context** | Customer names, internal service/host names, entity IDs, cost figures, real problem summaries | Confidential even when technically harmless |
| **Fixtures & screenshots** | Mock JSON captured from a live tenant, README screenshots with real entity names | The most common accidental leak |

**Entity IDs (`HOST-1A2B…`, `SERVICE-…`) and `event.id` values are tenant data.** They are
fine in a live app; they are not fine in a committed fixture, a README example, or a test.

---

## 2. The `.gitignore` baseline

Write this **before the first commit**, not after. Copy from
[`../assets/templates/gitignore.template`](../assets/templates/gitignore.template).

The Dynatrace-specific entries that a generic React `.gitignore` will miss:

```gitignore
# Dynatrace App Toolkit — contains cached OAuth tokens for your tenant
.dt-app/

# Environment variables (tenant URL, platform token)
.env
.env.local
.env.*.local
.env.production

# Certificates & keys
*.pem
*.key
*.cert
*.p12

# Local mock data captured from a real tenant
/settings/local-mock-data/secrets.json
/settings/local-mock-data/persistence/
**/fixtures/*.local.json

# Build & deploy artifacts
/dist
/out
/build
```

Verify it is actually working — an entry in `.gitignore` does nothing for a file Git is
**already tracking**:

```bash
git check-ignore -v .dt-app .env            # must print a matching rule for each
git ls-files | grep -Ei '\.env|\.dt-app|\.pem$|\.key$'   # must print nothing
```

If a sensitive file is already tracked:

```bash
git rm --cached -r .dt-app       # stop tracking, keep on disk
# then commit the removal — and treat the value as compromised: ROTATE IT.
```

---

## 3. The placeholder standard

Files that **must** be committed but reference the tenant are committed with placeholders.
Never with a real value, never with a "temporary" real value.

| File | Field | Committed value |
| --- | --- | --- |
| `app.config.json` | `environmentUrl` | `https://YOUR-ENVIRONMENT.apps.dynatrace.com/` |
| `.vscode/launch.json` | `url` | `https://YOUR-ENVIRONMENT.apps.dynatrace.com/...` |
| `.env.example` | `DT_ENVIRONMENT` | `https://YOUR-ENVIRONMENT.apps.dynatrace.com` |
| `.env.example` | `DT_PLATFORM_TOKEN` | `dt0s16.YOUR-PLATFORM-TOKEN` |
| `README.md` | any URL/ID | `YOUR-ENVIRONMENT`, `YOUR-TENANT-ID` |
| fixtures / tests | entity IDs | `HOST-0000000000000000`, `P-00000` |
| docs & screenshots | service names | `checkout-service`, `frontend`, `payments-api` |

**The placeholder must be obviously fake and obviously actionable.** `YOUR-ENVIRONMENT`
tells the reader to replace it. `abc12345` does not.

Every placeholder gets one line in the README saying exactly what to substitute and where
to find the value ("copy it from your Dynatrace address bar").

### The `.env` / `.env.example` pair

```bash
.env          # gitignored — real values live here, and only here
.env.example  # committed — same keys, placeholder values, plus a comment per key
```

`.env.example` is not optional. It is how a stranger clones the repo and knows what to set.

---

## 4. Pre-commit sanitize pass — mandatory

Run this **every time**, immediately before staging. Do not skip it because "I only
changed a component".

```bash
bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh
```

What it looks for (see the script for the exact patterns):

| Pattern | Catches |
| --- | --- |
| `dt0[a-z]\d{2}\.[A-Z0-9]{8,24}\.[A-Z0-9]{64}` | Dynatrace API / platform tokens |
| `https://[a-z]{3}\d{5}\.(apps\|live\|sprint)\.dynatrace\.com` | Real tenant URLs |
| `\b[a-z]{3}\d{5}\b` | Bare tenant IDs |
| `(HOST\|SERVICE\|PROCESS_GROUP\|APPLICATION)-[0-9A-F]{16}` | Real entity IDs |
| `[\w.+-]+@[\w-]+\.[\w.]+` | Emails (excluding the repo owner's own attribution line) |
| `(secret\|token\|password\|apikey\|api_key)\s*[:=]\s*['"][^'"]{8,}` | Generic hardcoded credentials |
| `-----BEGIN [A-Z ]*PRIVATE KEY-----` | Private keys |

**Scan the staged content, not just the working tree** — that is what is actually being
committed:

```bash
git diff --cached | bash skills/development-pattern-for-dynatrace/assets/scripts/scan-secrets.sh --stdin
```

**On any hit: stop.** Do not push. Replace with a placeholder, re-scan, then report to
the user what was found and what it was replaced with.

---

## 5. Defense in depth — three more layers

**a. ESLint catches secrets at edit time.** The project's `eslint.config.mjs` includes
`eslint-plugin-no-secrets` with a Dynatrace token regex. Keep it, and keep it at `"error"`:

```js
"noSecrets/no-secrets": ["error", {
  additionalRegexes: {
    "Dynatrace Token": "dt0[a-zA-Z]{1}[0-9]{2}\\.[A-Z0-9]{8,24}\\.[A-Z0-9]{64}",
  },
}],
```

**b. A git pre-commit hook makes it automatic.** Offer to install it; never install it
without asking. See [`../assets/templates/pre-commit.template`](../assets/templates/pre-commit.template).

**c. GitHub secret scanning + push protection.** For a public repo, both are free.
Recommend the user enables *Settings → Code security → Secret protection → Push protection*.
It is the last net, not the first.

---

## 6. Secrets at runtime — where they actually belong

Never in the bundle. The browser bundle is public to anyone who can open the app.

| Need | Correct mechanism |
| --- | --- |
| Call a Dynatrace platform API | Nothing to store — the platform injects the caller's identity. Declare the **scope** in `app.config.json`. |
| Call a third-party API needing a key | **App function** (server-side JS runtime) + **Credential Vault** (`environment-api:credentials:read`). The key never reaches the browser. |
| Local dev secrets | `.env`, gitignored, loaded by the toolkit — never imported into UI code |
| Per-user preference | App State / User App State service — not a secret store, but the right place for state |

**The browser cannot reach external domains at all.** The `connect-src` CSP directive is
not configurable for Dynatrace Apps. If a design calls for `fetch('https://api.vendor.com')`
from a React component, the design is wrong — route it through an app function.
→ [app-lifecycle.md § App functions](app-lifecycle.md)

`img-src` and `font-src` *are* configurable via custom CSP exceptions, if the app genuinely
needs an external image or font. Prefer bundling the asset instead.

---

## 7. Least privilege on scopes

Every scope in `app.config.json` is a permission the user is asked to grant. Each one is a
line item a security reviewer will question.

```jsonc
"scopes": [
  { "name": "storage:events:read", "comment": "DQL access to dt.davis.problems" },
  { "name": "state:app-states:read", "comment": "Read persisted acknowledgement state" }
]
```

- **One comment per scope, explaining the concrete feature that needs it.** A scope with
  no justification is a scope to delete.
- **Delete the template defaults you don't use.** `dt-app create` adds
  `storage:logs:read` and `storage:buckets:read`; if the app never queries logs, remove them.
- **Adding a scope after deployment forces every user to re-consent.** Get them right early.
- **Never add a write scope to enable a read feature.**

---

## 8. If a secret was already pushed

State it plainly to the user, immediately. Then, in this order:

1. **Rotate first.** Revoke the token in Dynatrace (*Settings → Platform tokens* /
   *Access tokens*) and issue a new one. Do this before touching Git — history rewriting
   takes minutes, and the token is exposed the whole time.
2. **Then clean history** (`git filter-repo`, or delete and recreate the repo if it is new
   and has no forks/stars). Warn that forks and caches may retain it regardless.
3. **Then prevent the recurrence** — add the pattern to the scanner, enable push protection.

Do not quietly fix it. The user needs to know a rotation is required.
