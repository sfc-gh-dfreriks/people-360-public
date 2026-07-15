# SAP BDC People 360 — Public (anonymous) build

A static, no-login public build of the SAP BDC People 360 dashboard, deployed to
GitHub Pages. Dashboards render from a **point-in-time data snapshot** (no Snowflake
connection, no credentials in the browser). The optional "Ask the Agent" page calls a
separate serverless Cortex agent when configured.

## How it works

- **Dashboards:** built with `VITE_STATIC=1`. The client reads pre-baked JSON in
  `public/data/*.json` instead of a live `/api`.
- **Filter combinations:** People 360 filters on **departments (12)** and
  **companies (3)**. A full power set is infeasible, so the snapshot bakes companies
  as a full power set × departments as {all + each single department} (104 combos).
  Company filtering is always exact; single-department and all-department views are
  exact; selecting **multiple** departments falls back to "all departments" (with the
  chosen companies). See `filterKey()` in `src/lib/api.ts`.
- **Data refresh:** re-run the exporter against the live dashboard server, commit the
  updated `public/data/*.json`, and push — Actions redeploys.
  ```bash
  # from the people_360_react monorepo, with the server running:
  EXPORT_BASE=http://localhost:3006 node scripts/export-static.mjs
  ```
- **Live agent (optional):** set the repo variable `AGENT_URL` to a deployed Cortex
  agent Worker URL. If unset, the Analyst page shows a "not available" notice but the
  dashboards work fully.

## Local build

```bash
npm ci
VITE_STATIC=1 npx vite build      # outputs dist/
python3 -m http.server -d dist    # preview
```

## Deploy

Push to `main` → `.github/workflows/deploy.yml` builds with
`BASE_PATH=/people-360-public/` and publishes `dist/` to GitHub Pages.

No secrets are stored in this repo. It contains only synthetic SAP demo data.
