# whitelabel-themes

Per-tenant brand themes served as static JSON. Consumer apps fetch a tenant's theme at runtime and pass it to the `lmnl-design-system` `ThemeProvider`. **Hosts validated JSON only** — no build step; the sole logic is schema validation in CI.

## Layout

```
themes/{tenantId}/theme.json     # one folder per tenant; self-contained full theme
themes/{tenantId}/assets/        # logo.svg, favicon.ico for that tenant
metadata/index.json              # lightweight listing of all tenants (id + name)
schema/v1.json                   # JSON Schema — the canonical contract
.github/workflows/validate.yml   # CI: schema + tenantId==folder gate on every PR
```

## Theme contract (schema v1)

Each `theme.json` is **self-contained** (the full token set). Keys are bare semantic names (no `--`); the design system applies them as `--{key}`.

```json
{
  "schemaVersion": 1,
  "tenantId": "acme-bank",
  "name": "Acme Bank",
  "colors": { "primary": "#2bb673", "background": "#ffffff" },
  "branding": { "logo": "logo.svg", "favicon": "favicon.ico" }
}
```

| Field | Required | Notes |
|---|---|---|
| `schemaVersion` | | `1` when present. Recommended. |
| `tenantId` | ✓ | Must equal the **folder name** and the app's `organisationId`. |
| `name` | ✓ | Human-readable brand name. |
| `colors` | ✓ | Bare-name → CSS value (hex or `var(--other)`). Keys must match real design-system tokens. |
| `typography` | | Same shape as `colors`. |
| `branding` | | `logo`/`favicon` as **filenames** relative to `assets/` (or absolute URLs). |

## Fetch URLs

```
theme:    https://<host>/themes/{tenantId}/theme.json
logo:     https://<host>/themes/{tenantId}/assets/{branding.logo}
listing:  https://<host>/metadata/index.json
```

## Onboarding a brand

1. Add `themes/{organisationId}/theme.json` (folder name **must** be the real `organisationId`) and its assets in `themes/{organisationId}/assets/`.
2. Include the full token set. **Accessibility:** verify text/background pairs meet WCAG AA.
3. Add the tenant to `metadata/index.json`.
4. Open a PR — CI validates the schema and that `tenantId` matches the folder. Merge only when green.

## Local validation

```bash
npx --yes -p ajv-cli@5 -p ajv-formats@3 ajv validate -c ajv-formats --spec=draft2020 --strict=false -s schema/v1.json -d "themes/*/theme.json"
```

> **Hosting note:** `raw.githubusercontent.com` works for a pilot but has no SLA and soft rate limits. Move to GitHub Pages / a CDN before production.
