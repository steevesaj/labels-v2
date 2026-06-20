# Isolation Label Generator

Browser-based generator for sequential electrical/mechanical isolation-point label sheets — print-ready PDF, QR asset tags, and batch manifest + QA exports. No install, no login, no backend.

A single self-contained HTML file. Pick an energy type and label style, enter a number range or specific tags, and it renders a positioned, print-ready PDF **entirely in the browser**. Built so field techs can self-serve from a SharePoint link.

**It generates:** a positioned PDF label sheet, an XLSX/CSV **manifest**, a fillable or flat **QA (Quality Assurance) proof sheet**, a **calibration sheet**, and an **Export all** that runs the selected outputs in sequence.

> **Current version: v4.5.0** (build 2026-06-20). See the [changelog](#changelog).

---

## Contents

- [For techs — how to use it](#for-techs--how-to-use-it)
- [For the admin — hosting](#for-the-admin--hosting)
- [For the admin — updating](#for-the-admin--updating)
- [Known limitations](#known-limitations)
- [Changelog](#changelog)
- [Calibration warning](#calibration-warning)

---

## For techs — how to use it

1. Open the tool (link on the SharePoint **Tools** page).
2. Pick the **Energy type** (Electrical, Gas, Pneumatic, Gravity, Water, Hydraulic) — this sets the header, hazard symbol, colour theme, and number prefix — and the **Company**, which sets the logo and default phone.
3. Choose **Mode → Range** (e.g. `5000`–`5200`) or **Tags** for reprints (e.g. `5042, 5108, 5511`).
4. *(Optional)* Open **Style** to switch the label layout (**Modern**, **OG**, **Classic**), the symbol set (**New** / **Old**), and the number **padding**.
5. *(Optional)* Toggle the **Site code**, **QR (Quick Response) code**, and **Asset ID** options for scannable, traceable tags.
6. Check the **live preview** (click it to zoom; use the arrows to step through labels) and the label/sheet count.
7. Click **Generate PDF** — it downloads a positioned sheet. The secondary buttons export a **Manifest** (XLSX/CSV), a **QA Proof Sheet**, a **Calibration sheet**, or **Export all** at once.
8. Print on the correct label stock. **Don't scale to fit — print at 100% / actual size.**

The tool remembers your last setup on that device. **Reset** (top-right of *Label setup*) clears it back to defaults.

---

## For the admin — hosting

The file is static, so any static host works. Pick one:

| Host | Notes |
|---|---|
| **GitHub Pages** | Free, fastest. Public URL — anyone with the link sees the page and embedded logo. Fine for non-sensitive use. |
| **Azure Static Web Apps** | Free tier; can stay anonymous or be gated to your org via Entra. Use if it must be internal-only. |
| **SharePoint link** | Host on one of the above, then add the URL as a link or Embed web part on the Tools page. Techs are already in M365, so no extra login. |

**Deploy steps (GitHub Pages):**
1. Rename the file to `index.html` and put it in a repo (suggested repo name: `label-generator`). Keep the `/assets` folder beside it.
2. Repo **Settings → Pages** → deploy from your branch. You get a URL like `youruser.github.io/label-generator/`.
3. Add that URL to the SharePoint Tools page.

> Note: don't upload the HTML *into* SharePoint and expect it to run — modern SharePoint blocks scripts in uploaded files. Host it, then link to it.

**Internet access:** the page loads three libraries from cdnjs (a public **CDN — Content Delivery Network**) at runtime — **jsPDF** (PDF), **qrcode-generator** (QR), and **ExcelJS** (XLSX manifest) — plus Google Fonts. Generation degrades gracefully if one fails (e.g. QR or XLSX warns and is skipped) but for a fully offline deployment, self-host those scripts beside the HTML and repoint the `<script src>` tags.

---

## For the admin — updating

Everything you'd change lives in the config block near the top of the `<script>` in the HTML.

- **Geometry / calibration (most important):** the `LAYOUTS` object defines every format **in inches** — `labelW`/`labelH`, margins (`mL`/`mT`), gaps (`gX`/`gY`), `cols`/`rows`, and an optional `radiusIn` (physical die-cut corner radius). If tags print misaligned, nudge these and re-test. **Always test-print on the real stock before rollout** — or use the built-in **Calibration sheet** export.
- **Colour themes:** the `THEMES` object — `color` drives the perimeter/header/footer (and number unless `num` is set), `bg` is the fill. Each energy type points at a theme.
- **Energy types:** the `ENERGY_TYPES` object — label, `prefix`, `pad`, default `company`, `theme`, and **two** symbol paths: `symbol` (Old) and `symbolNew` (New). Optional `symScale` fine-tunes one symbol's size. Add a type by adding an entry plus its SVGs.
- **Companies:** the `COMPANIES` object — `logo`, image `fmt` (`JPEG`/`PNG`), and default `phone`.
- **Logos / symbols:** stored in `/assets/logos` and `/assets/symbols`. Expected symbols are one **Old** + one **New** per energy type (e.g. `electrical.svg` / `electrical_new.svg`), and one logo per company (`pgs-logo.jpg`, `magna-logo.png`). To change one, replace the file (keep the filename, or edit the config path). A missing file shows a placeholder + hint instead of breaking. Non-electrical symbols are auto-tinted to the energy theme colour; SVGs are trimmed of transparent padding and rasterised at high resolution for crisp print.
- **Label styles:** three drawers render the tag art — `drawIsoModern` (default), `drawIsoOG`, and `drawIso` (Classic) — plus `drawCompact` for the small ID format. Fixed wording like "DO NOT REMOVE THIS TAG" lives in these functions.
- **New label types later:** the engine is config-driven — additional companies, energy types, or formats become config entries (the dropdowns, preview, PDF, manifest, and QA sheets all read from them), not a rewrite.

---

## Known limitations

A single reference for the gotchas noted throughout this README:

- **Print at 100% only.** Any scale-to-fit breaks registration against the die-cut stock (see [Calibration warning](#calibration-warning)).
- **QR runs are heavier.** Each label embeds a unique QR image (no de-duplication is possible), so large QR batches render slower and produce larger PDFs than plain runs.
- **Offline use needs self-hosting.** The page pulls three libraries from cdnjs plus Google Fonts at runtime. For a fully offline deployment, self-host those scripts beside the HTML and repoint the `<script src>` tags.
- **Missing assets degrade gracefully, not silently.** A missing logo or symbol shows a placeholder + hint rather than breaking the PDF — fix it before rollout.
- **SharePoint can't run the HTML directly.** Modern SharePoint blocks scripts in uploaded files; host the file on a static host and link to it.
- **Browser:** needs a current evergreen browser (Chrome, Edge, Firefox, Safari) with JavaScript enabled, plus internet access for the CDN libraries unless they are self-hosted.

---

## Changelog

### v4.5.0 — 2026-06-20
- **Two-column workspace.** Controls on the left, a sticky **Live preview** + **Sheet layout** diagram + run **stats** on the right, so the result stays in view while you work. Columns are height-balanced.
- **Three label styles** — **Modern** (default), **OG**, and **Classic** — switchable in a collapsible **Style** panel along with the symbol set and padding. The panel summarises the current pick (e.g. *Modern · New symbols · Pad 4*).
- **New vs Old symbol sets.** Every energy type ships two symbol artworks; **New** is the default. Symbols are trimmed to their real content and theme-tinted (electrical keeps its yellow/black ISO 7010 look) so they centre consistently across styles.
- **OnlineLabels OL996WJ (3" × 2", 10-up) is the default format,** verified against the official template to the twip (1/1440 inch) — size, sheet, margins, pitches, spacing, and the **3.18 mm (0.125") die-cut corner radius** are all wired to spec.
- **Batch export suite:** **Manifest** (XLSX with frozen header, auto-filter, and a *Print Status* dropdown / or CSV), **QA Proof Sheet** (fillable or non-fillable PDF), a **Calibration sheet**, and **Export all** to run the selected outputs in sequence.
- **Asset ID on the label** prints vertically along the right edge; its strip is reserved even when hidden, so the number and QR keep the same size and position whether or not it's shown.
- **Accessibility + polish:** keyboard focus rings on every control, **ARIA (Accessible Rich Internet Applications)** labels on the segmented groups, an SVG favicon + page metadata, click-to-zoom preview, and a **Reset to defaults** button in the *Label setup* header.
- Redundant on-screen label-number readouts removed (the number is legible on the preview itself).

> **Consolidates the v2.x–v4.x line** (after the v1.6.0 README): the asset-ID edge strip, per-style symbol handling, the OL996WJ format and spec-accurate corner radius, the manifest/QA/calibration exports, the two-column redesign, and the accessibility pass all landed across these releases.

### v1.6.0 — 2026-06-17
- **New asset ID format** — `{site}-{prefix}{number}-{YY}`, e.g. `YVR6-E-0007-26` (site code · energy type · asset number · last two digits of the year). This is the string the QR encodes. Previously `{year}-{site}-{prefix}{number}`.
- **Human-readable asset ID on the label** — when the QR is on, the asset ID also prints vertically along the right edge beside the QR, so the tag is identifiable by eye as well as by scan. The QR shifts in slightly to reserve the edge strip.
- **Asset ID breakdown under the preview** — shows the assembled ID with a labelled legend (site · energy · asset no. · year) so anyone can see what each part means.
- **Text now fits the space it's given** — the header, number, logo, and phone are measured and auto-sized to fit the available width on each format. This fixes crowding/overflow on the taller 3⅓" × 4" warning format when the QR is on, and protects against long site codes or wide numbers on any format.

### v1.5.0 — 2026-06-17
- **Site code toggle** — optional small line above the number (e.g., `YVR6`); the layout reflows when it's on or off. The site code field feeds both this line and the QR asset ID.
- **Scannable QR toggle** — the QR is unique per label, so when it's on the number/logo/phone recentre and the QR sits on the right. Empty year/site segments are dropped. QR generation uses `qrcode-generator` (cdnjs); if it fails to load, the toggle warns and is skipped rather than crashing.
- Both toggles apply to the full-tag formats (Isolation, Large warning, Custom-iso), not the small 1" ID label.
- Heads-up: QR runs are heavier than plain runs — each label embeds a unique QR image (no de-duplication possible), so large QR batches take longer and produce bigger files.

### v1.4.0 — 2026-06-17
- **Per-type colour themes** (`THEMES` config): the perimeter, header, number, and footer take the energy type's colour; the **phone number stays black**; the background can differ. Electrical = red on white (black number, per legacy), Gas = orange on white, Pneumatic = blue on white, Gravity = black on **yellow**, Water = teal on white, Hydraulic = blue on white. Symbol artwork keeps its own colours.
- New default phone numbers: Power Solutions Group `+1 657-502-6544`, Magna IV Engineering `+1 604-421-8020`.
- Default range is now **1–1000** (a full band).

### v1.3.0 — 2026-06-17
- **Label-format selector** (`LAYOUTS` config): Isolation 2"×4" 10-up (Avery 5163/8163, default), Large warning 3⅓"×4" 6-up (5164/8164), Small asset ID 1"×2⅝" 30-up (5160/8160), plus a **Custom sheet builder** (page Letter/A4, label W/H, columns/rows, margins, gaps, corner radius). Geometry is generic — label content scales to the box.
- **Six energy types** with confirmed prefix / pad / default brand: Electrical `E-` (pad 4), Gas `G-` (3), Pneumatic `P-` (3), Gravity `GR-` (3), Water `W-` (3), Hydraulic `H-` (4). Hydraulic defaults the brand to Magna IV; the others to PSG. Selecting a type also sets the default company (still overridable).
- **Responsive generation** (merged from the field-optimised build): progress bar + chunked render loop (`requestAnimationFrame`) so the UI never freezes, plus `compress:true` and a >1,500-label confirm.
- Retained **image aliasing** — each symbol/logo is embedded once and referenced, so a full 1,000-number band stays small and fast.

### v1.2.0 — 2026-06-17
- **Energy-type selector** (Electrical / Gas / Pneumatic / Gravity / Water) — changes the header text, the hazard symbol, and the default prefix together. Driven by the `ENERGY_TYPES` config at the top of the HTML; add a type by adding a config entry plus its `assets/symbols/<type>.svg` file.
- **Company selector** (Power Solutions Group / Magna IV Engineering) — swaps the logo and the default phone. Driven by the `COMPANIES` config.
- **Scale fix** — images are encoded once and reused via jsPDF aliasing, so large batches (a full 1,000-number band) render without crashing the tab.
- Graceful fallback: a missing symbol/logo file shows a placeholder and a "Missing asset file(s)" hint instead of breaking; the PDF still generates.

### v1.1.0 — 2026-06-17
- Real PSG logo and hazard-triangle symbol now pulled from `/assets` (relative paths) in place of the placeholder.
- Switched asset handling from base64-embed to repo-relative references — to swap an asset, replace the file in the repo (no code change).

### v1.0.0 — 2026-06-17
- Initial release. Isolation-point tags (proof of concept).
- Range and specific-tag (reprint) modes; editable prefix, padding, and phone.
- Client-side PDF, Avery 5163 (2"×4", 10-up); live preview; version stamped in UI and PDF metadata.

---

## Calibration warning

Print geometry must match the physical label stock exactly. Before any rollout: generate one sheet (or use the **Calibration sheet** export), print on the actual labels at **100% scale**, and confirm registration. Re-tune the layout's geometry if needed, then lock it.

> The default **OL996WJ** format has been cross-checked against the official OnlineLabels template (page size, margins, label cells, pitches, and the 3.18 mm die-cut corner) and matches to within sub-0.01 mm rounding. Still do a test print before a large run.
