# App lifecycle — scaffold, run, deploy

> Rule **R5** (deploy is a gate). Everything here targets a real tenant. Read the target
> aloud before you act on it.

---

## The toolkit

Dynatrace Apps are built with **`dt-app`** (the Dynatrace App Toolkit). Run it with `npx`
so the project's pinned version is used, not a global one.

**The command list below is a map, not a contract.** Verify against the installed version:

```bash
npx dt-app help
npx dt-app deploy --help
npx dt-app --version
```

| Command | Does |
| --- | --- |
| `npx dt-app create` | Scaffold a new app (prompts for name and environment URL) |
| `npx dt-app auth` | Browser OAuth against `environmentUrl`; caches tokens in `.dt-app/` |
| `npx dt-app dev` | Dev server with hot reload + reverse proxy to the tenant |
| `npx dt-app build` | Production build into `dist/` |
| `npx dt-app deploy` | Build **and** publish to the environment in `app.config.json` |
| `npx dt-app uninstall` | Remove the app from that environment |
| `npx dt-app update` | Update `@dynatrace*` packages and run migration codemods |
| `npx dt-app analyze` | Bundle analysis |
| `npx dt-app info` | CLI + environment diagnostics |
| `npx dt-app function create` (`f c`) | Generate an app function |
| `npx dt-app action create` | Generate a custom workflow action |
| `npx dt-app generate` | Scaffold app artifacts |
| `npx dt-app publish` | Publish toward the Dynatrace Hub (distribution, not tenant deploy) |

Mirror them as npm scripts so contributors do not need to know the CLI:

```json
{
  "scripts": {
    "start": "dt-app dev",
    "build": "dt-app build",
    "deploy": "dt-app deploy",
    "uninstall": "dt-app uninstall",
    "update": "dt-app update",
    "info": "dt-app info",
    "lint": "eslint ."
  }
}
```

---

## `app.config.json`

```jsonc
{
  "$schema": "./.dt-app/app.config.schema.json",
  "environmentUrl": "https://YOUR-ENVIRONMENT.apps.dynatrace.com/",   // ← placeholder in Git
  "app": {
    "name": "Command Center",
    "version": "1.1.3",
    "description": "Centralized problems management view for Dynatrace",
    "id": "my.command.center",
    "icon": "ui/assets/icons/app-icon.svg",
    "scopes": [
      { "name": "storage:events:read", "comment": "DQL access to dt.davis.problems" },
      { "name": "state:app-states:read",  "comment": "Read persisted ack state" },
      { "name": "state:app-states:write", "comment": "Persist ack/unack state" }
    ]
  }
}
```

Non-obvious rules:

- **`app.id` must start with `my.`** for unsigned apps. An id without that prefix requires
  digital app signing, and `deploy` will refuse. This is the most common first-deploy failure.
- **`environmentUrl` is a placeholder in the repository, always.** Developers point it at
  their own tenant locally. → [security-and-secrets.md](security-and-secrets.md)
- **Bump `version` on every deploy** you want distinguishable. It is what users see.
- **Every scope carries a `comment` justifying it.** Least privilege, and reviewable.
- **Adding a scope after release forces every user to re-consent** on next load.

---

## Local development

```bash
npm install
npx dt-app auth      # opens the browser; grants the declared scopes
npx dt-app dev
```

Things that trip people up:

- **Open the link the terminal prints — not `localhost:3000` directly.** The app must run
  inside the Dynatrace context or the platform SDKs have no identity and nothing works.
- The dev server proxies API calls to `environmentUrl` and injects the auth token.
- `dt-app dev` → *"Authentication failed"* almost always means the scopes in
  `app.config.json` changed since the last login. Re-run `npx dt-app auth`.
- *"invalid scopes"* means the tenant or the user lacks the IAM grants — a permissions
  problem in Dynatrace, not a code problem.
- Hot reload covers UI changes. Changes to `app.config.json` need a restart.

---

## Deploying ⛔ approval gate

```bash
npx dt-app auth      # if not authenticated, or if scopes changed
npx dt-app deploy
```

**Before running it, state the target and wait:**

> This will build and publish **`<App Name>` v`<version>`** to
> **`<environmentUrl>`**, making it available to every user of that tenant. Deploy?

Afterwards the app appears under *Dynatrace → Apps → `<App Name>`*. The local dev server
does not need to keep running.

### Deploying to a different environment

Deployment is per environment. There is no promote step.

1. Edit `environmentUrl` in `app.config.json` (locally — do not commit the real value).
2. `npx dt-app auth` — re-authenticate against the new tenant.
3. `npx dt-app deploy`.

Warn the user about what does **not** travel:

- **App State is per tenant.** Persisted state starts empty in the new environment. Nothing
  migrates.
- **Grail availability differs.** If the target lacks a table the app queries
  (e.g. `dt.davis.problems`), views render empty.
- **IAM differs.** The new tenant's admin must grant users the declared scopes.

---

## Beyond the UI

| Need | Mechanism | Notes |
| --- | --- | --- |
| Call an external API | **App function** (`dt-app function create`) | The browser cannot: `connect-src` CSP is not configurable. Server-side JS runtime, low-latency access to Grail and platform APIs. |
| Use a third-party API key | App function + **Credential Vault** | Scope `environment-api:credentials:read`. Never in the bundle. |
| Extend Workflows | **Custom action** (`dt-app action create`) | App function + UI component, selectable as a workflow task. |
| Navigate to another app | **Intents** / Navigation SDK | Declare receivable intent types in the app config. Not `<a href>`. |
| Persist app-wide state | **App State** service (`@dynatrace-sdk/client-state`) | Shared across tenant users. **400 KB total limit** — plan cleanup. |
| Persist per-user state | **User App State** | Same service, user-scoped. |
| Store structured documents | **Document** service | Dashboards, notebooks, shareable JSON. |
| Read user context | `@dynatrace-sdk/app-environment` | `getCurrentUserDetails()`, env IDs, URLs. |
| Read theme / locale / timezone | `@dynatrace-sdk/user-preferences` | Read-only. Custom settings go in App State. |
| Format units | `@dynatrace-sdk/units` | Bytes → MiB, ms → s, consistently. |

App functions run in the Dynatrace JavaScript runtime with real limits (execution time,
memory, available Node APIs). Read `dt-js-runtime` from the official Dynatrace skills
before writing one. → [dql-and-mcp.md](dql-and-mcp.md)

---

## Upgrades

```bash
npx dt-app update      # bumps @dynatrace* packages and applies codemods
npx tsc --noEmit && npm run lint && npx dt-app build
```

Strato ships **migration codemods** — run them rather than hand-editing renamed props.
Read the changelogs when something moves:
`https://developer.dynatrace.com/release-notes/design-system/components-changelog/`.

`@typescript-eslint/no-deprecated` is set to `"error"` in the recommended config precisely
so deprecations surface at build time instead of at the next breaking release.

---

## Deploy troubleshooting

| Symptom | Cause & fix |
| --- | --- |
| *"app must be signed"* | `app.id` doesn't start with `my.` |
| *"Authentication failed"* on `dev`/`deploy` | Scopes changed since last login → `npx dt-app auth` |
| *"invalid scopes"* | Tenant/user lacks IAM grants for a declared scope |
| Table shows *"No data available"* | Grail not enabled, or the queried table doesn't exist here |
| `404 Unknown key: <state-key>` in logs | Expected on a fresh tenant — no state written yet. Handle it as "empty", not as an error. |
| App loads blank in Dynatrace | Opened `localhost:3000` directly instead of the printed link |
| Deploy succeeds, app not in the menu | Wrong `environmentUrl`, or the user lacks permission to see it |
