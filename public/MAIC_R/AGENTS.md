# MAIC_R — Project Memory

## What this project is

A Quarto **revealjs** presentation template ("InnoCare Presentation Style") plus a
slide deck introducing the R package `maicplus`. This directory is a presentation
project, not a data-analysis project — there is no raw dataset to load.

## Directory structure

- `maic.qmd` — original blank InnoCare revealjs template (SAS/R code highlighting demo, title/end slide).
- `maicplus-intro.qmd` — revealjs deck introducing the `maicplus` R package (MAIC methodology, workflow, live code demos). Renders to `maicplus-intro.html`.
- `_extension.yml` / `_extensions/` — Quarto extension config; includes `quarto-ext/pointer` (used via `revealjs-plugins: [pointer]`) and `quarto-ext/fontawesome`.
- `custom.scss` — base InnoCare theme (primary color `#4d8dc9`).
- `ending.scss` — end-slide styling (large title, orange `#FF6600` text).
- `styles/` — ~96 highlight.js CSS themes for code syntax highlighting; template currently uses `styles/vs.css`.
- `highlight.pack.js` — highlight.js library, loaded via `include-in-header` for SAS code chunk highlighting (`{class='SAS'}`).
- `revealjs-assets/title-slide.html`, `revealjs-assets/end-slide.html` — custom HTML partials wired in via `template-partials`.
- `assets/*.png` — branding images (`footlogo.png` for `logo:`, `logo-title.png`, `logo-end.png`, `titlelogo.png`, `endlogo.png`, `headlogo.png`).

## Standard YAML header pattern for new decks

```yaml
format:
  revealjs:
    theme: [default, ending.scss]
    smaller: true
    scrollable: true
    chalkboard: true
    logo: assets/footlogo.png
    include-in-header:
      - text: |
          <script src="highlight.pack.js"></script>
          <script>hljs.initHighlightingOnLoad();</script>
    css: styles/vs.css
    template-partials:
        - revealjs-assets/title-slide.html
revealjs-plugins:
  - pointer
execute:
  echo: true
  eval: false
```
End slide convention: `# Thanks {.end-slide}` followed by
`![](assets/logo-end.png){.absolute top=200 left=330 width="400" height="320"}`.

Chapter divider pages: when adding a new chapter/section divider (e.g. the case
study opener), do NOT use a raw `#`/`##` header or the default title display.
Copy the Case study divider page exactly, using the `appendix-title` container
pattern (styled in `title-style.scss`):

```html
<div class="appendix-title-container">
  <div class="appendix-title">Case study: An unanchored MAIC in relapsed/refractory mantle cell lymphoma</div>
</div>
```

SAS code chunks use `` ```{class='SAS'} `` for highlight.js styling instead of a real
executable knitr engine.

Mermaid gotcha (verified on Quarto 1.9.38): mermaid diagrams must use
`` ```{mermaid} `` chunks — a plain `` ```mermaid `` fence only renders in
preview, not in the built revealjs HTML. Also, global `execute: echo: true`
/ `eval: false` both break mermaid: `eval: false` silently skips diagram
generation (empty figure boxes) and `echo: true` shows the mermaid source as
plain code blocks instead of diagrams. Keep both out of the header and set
`#| echo: true` / `#| eval: false` on R chunks individually (mermaid chunks
do not parse `#|` chunk options in this Quarto version). Diagrams render
client-side via `maicplus-intro_files/libs/quarto-diagram/mermaid.min.js`.

## `maicplus` package (used in maicplus-intro.qmd)

- Installed locally, CRAN version 0.1.2. Implements Matching-Adjusted Indirect
  Comparison (Signorovitch et al. 2012); maintained by MSD/Roche
  (github.com/hta-pharma/maicplus).
- Core workflow: `center_ipd()` → `estimate_weights()` → `check_weights()` /
  `plot_weights_ggplot()` → `maic_anchored()` / `maic_unanchored()` →
  `kmplot()`/`kmplot2()`.
- Built-in demo datasets used in the slides: `adsl_sat`, `adtte_sat`,
  `pseudo_ipd_sat` (unanchored/"sat" scenario, single-arm-style comparison);
  `adsl_twt`, `adtte_twt`, `pseudo_ipd_twt` (anchored/"twt" scenario, has common
  comparator arm `C`).
- Package vignettes worth pulling from for more content: `introduction`,
  `calculating_weights`, `anchored_survival`, `unanchored_survival`,
  `anchored_binary`, `unanchored_binary`, `kaplan_meier_plots`.
- Gotcha: `plot_weights_ggplot()` requires `bin_col`, `vline_col`, `main_title`,
  and `bins` — none have defaults.

## Real-world case study (in maicplus-intro.qmd)

`maicplus-intro.qmd` includes a verified real case study, separate from the
package's toy demo data:

- **Project location** (outside this repo):
  `C:\Users\xiangt\Documents\01_Studies\MAIC-102+107\acala\` — an unanchored
  MAIC comparing **orelabrutinib (ICP-022, pooled studies 102+107, IPD, N=129)**
  vs. **acalabrutinib (ACE-LY-004, aggregate data, N=124)** in relapsed/refractory
  mantle cell lymphoma, for Beijing Innocare Pharma Tech Co., Ltd.
- Scripts: `maic-start.R` (shared setup: ADaM read-in, 16 matching covariates,
  `center_ipd()`/`estimate_weights()`), `maic-t1-baseline.R` (covariate balance),
  `maic-t2-binary.R` (ORR/CRR via `maic_unanchored(endpoint_type="binary")`),
  `maic-t3-survival.R` (PFS/OS via `endpoint_type="tte"`), `maic-t4-ae.R` (looped
  TEAE analysis), `maic-f1/f2-km-*.R` (`kmplot2()` figures). ADaM source data
  lives one level up in `../adam/*.sas7bdat`; comparator AgD/pseudo-IPD in
  `acala_*.csv` files. Outputs are RTF (via officer/flextable), logged with `logrx`.
- Key verified results (re-run directly, not just read from logs): ESS drops
  from 129 to **50.2** (−61%); ORR OR 1.05→0.76 unadjusted→adjusted; CRR OR
  0.70→0.64; PFS HR 0.83→1.03; OS HR 0.88→1.21 — adjustment moves all efficacy
  endpoints toward/past null.
- Real diagnostic plots generated from this project and saved into
  `assets/acala_weights.png` (weight distribution) and `assets/acala_km_pfs.png`
  (PFS KM curves) for use in the slides — regenerate these by re-sourcing
  `maic-start.R` from the `acala` project directory if the underlying data changes.
- The slide version of the KM plot is anonymized: call `kmplot2()` with
  `trt_ipd = "Intervention"` AND relabel the IPD `ARM` column to `"Intervention"`
  first (kmplot2 filters the IPD data by `ARM == trt_ipd`). The project's own
  `maic-f1-km-pfs.R` still uses `"ICP-022"` for its RTF output.
- Slides also cover the project's reporting/audit tooling: `officer` +
  `flextable` for RTF tables/figures (`set_header_labels()`/`add_header_row()`
  for the "Without/After adjustment" column groups, `block_list()`-built
  `header_default`/`footer_default` reused across all scripts, `save_as_rtf()`),
  and `logrx::axecute()` (called from `maic-batch-with-log.R`) for
  reproducibility logs — each `<script>.log` records logrx metadata, user/file
  hash, full session info (R version + every package version), a per-expression
  execution trace, and the script's final output path. `Batchrun_R.bat` is a
  non-interactive `Rscript.exe` alternative for batch runs.
- Verified the `acala` project directory has **no `renv.lock`/`renv/` folder**
  (checked directly) — the slides include a `renv` slide framed as a
  recommended complement to `logrx` (pins exact package versions via
  `renv::snapshot()`/`restore()`) rather than something already in use there.

## Rendering

`quarto render <file>.qmd` from this directory works and has been verified for
both `maic.qmd`-style templates and `maicplus-intro.qmd`.
