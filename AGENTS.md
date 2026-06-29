# AGENTS.md

## Cursor Cloud specific instructions

### What this repo is
This is a **static website** (the "UAGC Strategic Intelligence Suite") deployed to GitHub Pages. It is a collection of standalone HTML reports/dashboards plus supporting CSV/PDF/Markdown data. There is **no build step, no package manager, no dependencies, and no automated test suite**. The GitHub Actions workflow `.github/workflows/static.yml` simply uploads the entire repo root as the Pages artifact.

### Running it locally (development)
Serve the repository root with any static file server. The environment already has Python 3 and Node available, so no install step is needed:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000/index.html` (the central dashboard). Internal links are relative to the repo root, so the server must be started from `/workspace`.

- Entry point: `index.html` (central hub linking to all reports).
- Local reports include `seo-cro-audit-uagc.html`, `Reputation management/uagc-comprehensive-reputation-analysis.html`, `cookie/UAGC-cookie-personalization.html`, `Discovery-Awareness/federal-student-aid-changes-2025.html`, `reddit/UAGC-Reddit-Brand-Perception-Report.html`.
- Some dashboard links point to **external** GitHub Pages sites (e.g. `omac049.github.io/...`) and require internet access; these are expected to open in a new tab.

### Lint / test / build
- There is **no lint, test, or build** tooling configured. Do not invent one unless asked.
- Validate changes by serving locally and viewing the affected HTML page in a browser.
- Note: directory names contain spaces (e.g. `Reputation management/`), so URL-encode spaces (`%20`) when testing with `curl`.
