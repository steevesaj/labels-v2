# Isolation Label Generator

Browser-based generator for sequential electrical isolation-point label sheets — Avery 5163, 10-up, no install or login.

A single self-contained HTML file. Enter a number range or specific tags, and it renders a print-ready PDF **entirely in the browser** — no server, no backend, no software to install. Built so field techs can self-serve from a SharePoint link.

---

## For techs — how to use it

1. Open the tool (link on the SharePoint **Tools** page).
2. Choose **Number range** (e.g. `5000` to `5200`) or **Specific tags** for reprints (e.g. `5042, 5108, 5511`).
3. Check the live preview and the label/sheet count.
4. Click **Generate PDF** — it downloads a positioned sheet.
5. Print on the correct label stock (Avery 5163, 10 per sheet). Don't scale to fit — print at 100% / actual size.

---

## For the admin — hosting

The file is static, so any static host works. Pick one:

| Host | Notes |
|---|---|
| **GitHub Pages** | Free, fastest. Public URL — anyone with the link sees the page and embedded logo. Fine for non-sensitive use. |
| **Azure Static Web Apps** | Free tier; can stay anonymous or be gated to your org via Entra. Use if it must be internal-only. |
| **SharePoint link** | Host on one of the above, then add the URL as a link or Embed web part on the Tools page. Techs are already in M365, so no extra login. |

**Deploy steps (GitHub Pages):**
1. Rename the file to `index.html` and put it in a repo (suggested repo name: `label-generator`).
2. Repo **Settings → Pages** → deploy from your branch. You get a URL like `youruser.github.io/label-generator/`.
3. Add that URL to the SharePoint Tools page.

> Note: don't upload the HTML *into* SharePoint and expect it to run — modern SharePoint blocks scripts in uploaded files. Host it, then link to it.

---

## For the admin — updating

Everything you'd change lives at the top of the `<script>` block in the HTML.

- **Geometry / calibration (most important):** the `MARGIN_*`, `GUTTER`, and `LABEL_*` constants are points (72 pt = 1 inch), set to Avery 5163. If tags print misaligned, nudge these and re-test. **Always test-print on the real stock before rollout.**
- **Logos / symbols:** stored in `/assets/logos` and `/assets/symbols`, referenced from the `ENERGY_TYPES` (symbols), `COMPANIES` (logos), and `LAYOUTS` (formats) config objects at the top of the HTML. The tool expects one symbol per energy type: `electrical.svg`, `gas.svg`, `pneumatic.svg`, `gravity.svg`, `water.svg`, `hydraulic.svg`, and one logo per company: `pgs-logo.jpg`, `magna-logo.png`. To change one, replace the file (keep the filename, or edit the config path). To add a type, company, or format, add a config entry — the dropdowns, preview, and PDF all read from the config. A missing file shows a placeholder + hint instead of breaking. Symbol SVGs are rasterized at high resolution for the PDF.
- **Fixed text / phone:** phone and prefix are editable in the UI; the header and footer text are in the `drawLabel()` function.
- **New label types later:** the engine is structured so additional companies/label types become config, not a rewrite.

---

## Versioning

Semantic Versioning — `MAJOR.MINOR.PATCH`:

- **PATCH** (`1.0.x`) — geometry tweaks, logo swap, copy fixes.
- **MINOR** (`1.x.0`) — new label type/company, new field or option.
- **MAJOR** (`x.0.0`) — structural redesign or any breaking change.

The version is the single source of truth in one constant (`VERSION`) and surfaces in three places:
1. The **header badge** in the UI (`v1.3.0`).
2. The **PDF metadata** of every generated sheet (creator + keywords include the version, the generation date, and the tag range) — so any printed sheet is traceable to the exact build and parameters that produced it.
3. The **changelog** below.

**To release a new version:** update `VERSION` and `BUILD_DATE` at the top of the file, add a changelog entry, and (if using git) tag the commit `vX.Y.Z`.

### Changelog

#### v1.3.0 — 2026-06-17
- **Label-format selector** (`LAYOUTS` config): Isolation 2"×4" 10-up (Avery 5163/8163, default), Large warning 3⅓"×4" 6-up (5164/8164), Small asset ID 1"×2⅝" 30-up (5160/8160), plus a **Custom sheet builder** (page Letter/A4, label W/H, columns/rows, margins, gaps, corner radius). Geometry is generic — label content scales to the box.
- **Six energy types** with confirmed prefix / pad / default brand: Electrical `E-` (pad 4), Gas `G-` (3), Pneumatic `P-` (3), Gravity `GR-` (3), Water `W-` (3), Hydraulic `H-` (4). Hydraulic defaults the brand to Magna IV; the others to PSG. Selecting a type also sets the default company (still overridable).
- **Responsive generation** (merged from the field-optimized build): progress bar + chunked render loop (`requestAnimationFrame`) so the UI never freezes, plus `compress:true` and a >1,500-label confirm.
- Retained **image aliasing** — each symbol/logo is embedded once and referenced, so a full 1,000-number band stays small and fast.

#### v1.2.0 — 2026-06-17
- **Energy-type selector** (Electrical / Gas / Pneumatic / Gravity / Water) — changes the header text, the hazard symbol, and the default prefix together. Driven by the `ENERGY_TYPES` config at the top of the HTML; add a type by adding a config entry plus its `assets/symbols/<type>.svg` file.
- **Company selector** (Power Solutions Group / Magna IV Engineering) — swaps the logo and the default phone. Driven by the `COMPANIES` config.
- **Scale fix** — images are encoded once and reused via jsPDF aliasing, so large batches (a full 1,000-number band) render without crashing the tab.
- Graceful fallback: a missing symbol/logo file shows a placeholder and a "Missing asset file(s)" hint instead of breaking; the PDF still generates.

#### v1.1.0 — 2026-06-17
- Real PSG logo and hazard-triangle symbol now pulled from `/assets` (relative paths) in place of the placeholder.
- Switched asset handling from base64-embed to repo-relative references — to swap an asset, replace the file in the repo (no code change).

#### v1.0.0 — 2026-06-17
- Initial release. Isolation-point tags (proof of concept).
- Range and specific-tag (reprint) modes; editable prefix, padding, and phone.
- Client-side PDF, Avery 5163 (2"×4", 10-up); live preview; version stamped in UI and PDF metadata.

---

## Calibration warning

Print geometry must match the physical label stock exactly. Before any rollout: generate one sheet, print on the actual labels at **100% scale**, and confirm registration. Re-tune the geometry constants if needed, then lock it.
