# ave-site

Website for the [AVE (Agentic Vulnerability Enumeration)](https://github.com/bawbel/ave) open standard.

**Live site:** [ave.bawbel.io](https://ave.bawbel.io)

---

## What this repo is

This repo contains the website only — HTML, CSS, JS, and assets for `ave.bawbel.io`. It is a consumer of the AVE standard, not part of the standard itself.

The records, schema, detection rules, and fixtures live in **[bawbel/ave](https://github.com/bawbel/ave)**. Changes to AVE records automatically trigger a rebuild and deployment of this site via GitHub Actions.

---

## Pages

| Page | URL | What it does |
|---|---|---|
| Landing | `/` | Overview, stats, links to all sections |
| Registry | `/registry.html` | Searchable, filterable table of all AVE records |
| Crosswalks | `/crosswalks.html` | How other scanners and frameworks map to AVE ids |
| Architecture | `/architecture.html` | How an AVE record works — for implementers |
| Schema | `/schema.html` | Every field, type, required/optional, meaning |

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

# serve locally (any static server)
npx serve .
# or
python3 -m http.server 8080
```

Open [http://localhost:8080](http://localhost:8080).

### Build commands

| Command | What it does |
|---|---|
| `npm run build:local` | Build from `../ave/records` (local dev) |
| `npm run build` | Build from default path (CI) |
| `npm run build:drafts` | Include `status: "draft"` records |
| `npm run build:dry` | Validate records without writing `records.js` |

### Passing a custom records path

```bash
node scripts/build-records.js \
  --records-dir /path/to/ave/records \
  --schema      /path/to/ave/schema/ave-record.schema.json \
  --out         records.js
```

---

## How the build works

`scripts/build-records.js` reads every `AVE-YYYY-NNNNN.json` file from the records directory, validates each one against the AVE JSON schema (requires `ajv` and `ajv-formats`), excludes drafts, sorts by severity then id, and writes `records.js` — the static JS module the site loads.

`records.js` is **gitignored** in this repo. It is generated fresh on every CI build from the canonical source in `bawbel/ave`. Never edit it by hand.

---

## Deployment

The site deploys to [GitHub Pages](https://docs.github.com/en/pages) via `.github/workflows/deploy.yml`.

**Triggers:**
- Any push to `main` in this repo
- A `repository_dispatch` event (`ave-records-updated`) sent by `bawbel/ave` when records, schema, or rules change
- Manual trigger from the Actions tab

**To wire up the cross-repo trigger:**
1. Create a fine-grained PAT with `bawbel/ave-site` Contents (read) + Actions (write) permissions
2. Add it as `AVE_SITE_DEPLOY_TOKEN` in `bawbel/ave` → Settings → Secrets
3. Copy `notify-ave-site.yml` into `bawbel/ave/.github/workflows/`

After that, pushing a new AVE record to `bawbel/ave` automatically rebuilds and deploys the site within a few minutes.

**Custom domain:** Set `ave.bawbel.io` as the custom domain in the `bawbel/ave-site` GitHub Pages settings. Add a CNAME record pointing to `bawbel.github.io`.

---

## Repository structure

```
ave-site/
├── index.html              Landing page
├── registry.html           Searchable AVE registry
├── crosswalks.html         Scanner and framework crosswalks
├── architecture.html       Implementer guide
├── schema.html             Schema v1.0.0 reference
├── styles.css              Shared stylesheet
├── records.js              GENERATED — do not edit (gitignored)
├── manifest.json           PWA manifest
├── robots.txt
├── sitemap.xml
├── favicon.ico
├── ave-favicon.svg
├── apple-touch-icon.png
├── ave-favicon-192.png
├── ave-favicon-512.png
├── scripts/
│   └── build-records.js    Generates records.js from bawbel/ave
├── notify-ave-site.yml     Copy this to bawbel/ave/.github/workflows/
└── .github/
    └── workflows/
        └── deploy.yml      Build + deploy to GitHub Pages
```

---

## Contributing

This repo is for the website. To propose a new AVE record or a schema change, open an issue or PR in **[bawbel/ave](https://github.com/bawbel/ave)**.

To fix a bug or improve the site (UI, crosswalk tables, content), open a PR here.

---

## Related

- **[bawbel/ave](https://github.com/bawbel/ave)** — the AVE standard (records, schema, rules)
- **[bawbel/scanner](https://github.com/bawbel/scanner)** — reference implementation of AVE
- **[bawbel.io](https://bawbel.io)** — Bawbel

---

## License

Apache 2.0 — the same license as the AVE standard itself.
