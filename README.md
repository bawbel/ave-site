<div align="center">

<img src="https://ave.bawbel.io/ave-logo-full.svg" alt="AVE — Agentic Vulnerability Enumeration" width="320" />

<br/>
<br/>

**Website for the [AVE](https://github.com/bawbel/ave) open standard — the CVE for AI agents.**

[![Live site](https://img.shields.io/badge/live-ave.bawbel.io-0a3024?style=flat-square&logo=googlechrome&logoColor=d4a017)](https://ave.bawbel.io)
[![Deploy](https://img.shields.io/github/actions/workflow/status/bawbel/ave-site/deploy.yml?style=flat-square&label=deploy&color=0f6e56)](https://github.com/bawbel/ave-site/actions/workflows/deploy.yml)
[![AVE standard](https://img.shields.io/badge/standard-bawbel%2Fave-0a3024?style=flat-square)](https://github.com/bawbel/ave)
[![Records](https://img.shields.io/badge/records-48-critical?style=flat-square&color=0f6e56)](https://ave.bawbel.io/registry.html)
[![Schema](https://img.shields.io/badge/schema-v1.0.0-d4a017?style=flat-square)](https://ave.bawbel.io/schema.html)
[![License](https://img.shields.io/badge/license-Apache%202.0-green?style=flat-square)](LICENSE)

</div>

---

## What this repo is

This repo contains the website only — HTML, CSS, JS, and assets for `ave.bawbel.io`. It is a **consumer** of the AVE standard, not part of the standard itself.

The records, schema, detection rules, and fixtures live in **[bawbel/ave](https://github.com/bawbel/ave)**. Pushing new AVE records to that repo automatically triggers a rebuild and deployment of this site via GitHub Actions `repository_dispatch`.

---

## Pages

| Page | URL | What it does |
|---|---|---|
| Landing | `/` | Overview, stats, links to all sections |
| Registry | `/registry.html` | Searchable, filterable table of all AVE records with full detail drawer |
| Crosswalks | `/crosswalks.html` | How SkillSpector, ClawScan, OWASP MCP, and MITRE ATLAS map to AVE ids |
| Architecture | `/architecture.html` | How an AVE record works — for tool implementers |
| Schema | `/schema.html` | Every field, type, required/optional, meaning — schema v1.0.0 reference |

---

## Important notes

### Record shape compatibility

AVE records at `schema_version: 0.2.0` store severity at `aivss.aivss_severity`
rather than the top-level `severity` field required by schema v1.0.0. The registry
page includes a runtime normaliser that backfills missing fields automatically.
This is resolved permanently when records are migrated to schema v1.0.0 in v1.1.

### Schema validation flag

Until all 48 records are migrated to v1.0.0, run the build with `--skip-validation`:

```bash
npm run build:skip-validation
```

Remove this flag after the v1.1 migration is complete. The validator is intentionally
strict — if it rejects a record, the record needs to be fixed, not the schema.

---

## Local development

**Prerequisites:** Node.js 18+, and `bawbel/ave` checked out as a sibling directory.

```
parent/
├── ave/          ← github.com/bawbel/ave
└── ave-site/     ← github.com/bawbel/ave-site  (this repo)
```

```bash
git clone https://github.com/bawbel/ave
git clone https://github.com/bawbel/ave-site
cd ave-site

npm install
npm run build:local   # reads ../ave/records → writes records.js

# serve locally
npx serve .
# or
python3 -m http.server 8080
```

Open [http://localhost:8080](http://localhost:8080).

### Build commands

| Command | What it does |
|---|---|
| `npm run build:local` | Build from `../ave/records` (local dev, validates against schema) |
| `npm run build:skip-validation` | Build without schema validation (use until v1.1 migration) |
| `npm run build` | Build from default path (CI) |
| `npm run build:drafts` | Include `status: "draft"` records |
| `npm run build:dry` | Validate records without writing `records.js` |

### Custom path

```bash
node scripts/build-records.js \
  --records-dir ../ave/records \
  --schema      ../ave/schema/ave-record-1.0.0.schema.json \
  --out         records.js \
  --skip-validation   # remove after v1.1 migration
```

---

## How the build works

`scripts/build-records.js` reads every `AVE-YYYY-NNNNN.json` from the records
directory, validates against the schema (skippable via `--skip-validation`),
excludes drafts, sorts CRITICAL → HIGH → MEDIUM → LOW then by id, and writes
`records.js` — the static JS module the site loads at runtime.

`records.js` is **gitignored**. It is generated fresh on every CI build from
the canonical source in `bawbel/ave`. Never edit it by hand.

---

## Deployment

Deploys to [GitHub Pages](https://docs.github.com/en/pages) via `.github/workflows/deploy.yml`.

**Triggers:**
- Push to `main` in this repo
- `repository_dispatch` (`ave-records-updated`) from `bawbel/ave` when records, schema, or rules change
- Manual trigger from the Actions tab

**To wire up the cross-repo trigger:**

1. Create a fine-grained PAT — repository `bawbel/ave-site`, permissions: Contents (read) + Actions (write)
2. Add it as `AVE_SITE_DEPLOY_TOKEN` in `bawbel/ave` → Settings → Secrets
3. Copy `notify-ave-site.yml` into `bawbel/ave/.github/workflows/`

After that, pushing a new AVE record to `bawbel/ave` automatically rebuilds and
deploys the site within a few minutes.

**Custom domain:** Set `ave.bawbel.io` as the custom domain in GitHub Pages settings.
Add a CNAME record in Namecheap pointing `ave` → `bawbel.github.io`.

---

## Repository structure

```
ave-site/
├── index.html              Landing page
├── registry.html           Searchable AVE registry
├── crosswalks.html         Scanner and framework crosswalks
├── architecture.html       Implementer guide with SVG diagrams
├── schema.html             Schema v1.0.0 field reference
├── styles.css              Shared stylesheet (all pages)
├── records.js              GENERATED — gitignored, never edit by hand
├── CNAME                   Custom domain: ave.bawbel.io
├── manifest.json           PWA manifest
├── robots.txt
├── sitemap.xml
├── favicon.ico             Multi-size (16/32/48px) — transparent stingray
├── ave-favicon.svg         SVG favicon — transparent background
├── apple-touch-icon.png    180px — iOS home screen (dark background)
├── ave-favicon-192.png     Android home screen
├── ave-favicon-512.png     PWA / OG image
├── scripts/
│   └── build-records.js    Generates records.js from bawbel/ave records
├── notify-ave-site.yml     ← copy to bawbel/ave/.github/workflows/
└── .github/
    └── workflows/
        └── deploy.yml      Build + deploy to GitHub Pages
```

---

## SEO and performance

- Unique `<title>` and `<meta description>` per page
- Open Graph + Twitter card on all pages
- JSON-LD structured data (WebSite schema with SearchAction)
- `sitemap.xml` and `robots.txt`
- PWA manifest with theme colour `#0a3024`
- Responsive to 375px (iPhone SE)
- Static files only — no backend, no runtime JS dependencies, no CDN for functionality
- Google Fonts loaded via preconnect for IBM Plex Sans / Mono + Playfair Display

---

## Contributing

This repo is for the website. To propose a new AVE record or a schema change, open an issue in **[bawbel/ave](https://github.com/bawbel/ave)**.

To fix a bug or improve the site (UI, crosswalk tables, content), open a PR here. Keep the site a consumer of the standard — no AVE record data lives here except what `build-records.js` generates.

---

## Related

| | |
|---|---|
| [bawbel/ave](https://github.com/bawbel/ave) | The AVE standard — records, schema, rules |
| [bawbel/scanner](https://github.com/bawbel/scanner) | Reference implementation of AVE |
| [api.piranha.bawbel.io](https://api.piranha.bawbel.io) | Threat intel API |
| [bawbel.io](https://bawbel.io) | Bawbel |

---

## License

Apache 2.0 — the same license as the AVE standard itself.