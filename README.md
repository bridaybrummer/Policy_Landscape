# PhD_DataQuality_effects_HealthOutcomes

A `{targets}`-orchestrated R research pipeline investigating whether South
African CRVS (civil registration and vital statistics) data quality distorts
provincial/national health budget allocation for causes outside HIV/TB.

**This is a pipeline for a question that hasn't been answered yet** — see
[docs/PLAN.md](docs/PLAN.md) for the hypothesis and working analysis plan
(provisional, revised freely), and [docs/KNOWLEDGE.md](docs/KNOWLEDGE.md) for
established facts and confirmed data sources (updated only when a source is
confirmed or a figure corrected). Keep the two registers separate — a
funding-gap claim belongs in PLAN.md until it's actually demonstrated.

For a conceptual, conference-facing introduction to this work (draft, not
yet submitted), see [site/chapters/06-cphia-abstract.qmd](site/chapters/06-cphia-abstract.qmd)
(the canonical, working draft — there is no separate markdown copy).

## Status

Two real, working data pulls exist (see `scripts/run_first_implementer.R`):

- `fetch_vulekamali()` — real provincial Health-department budget data from
  National Treasury's EPRE dataset (CSV/XLSX download, discovered per
  release year; verified with a live pull of 42,578 rows across FY2020-2027,
  all 9 provinces).
- `R/extract_app_citations.R` — downloads real National DoH strategic
  documents (APP, Strategic Plans) and extracts citations of known data
  sources (Stats SA, GBD, SADHS, GHS, etc.), producing a citation trail.

Every other data-fetcher is still a stub: each one either aborts loudly and
specifically (until an access method is confirmed; see `docs/KNOWLEDGE.md`
"Open questions") or documents a manual-download workflow for sources that
require registration or block automated access (GBD, DHB, SADHS microdata,
NCoDV).

## Structure

```
R/                  one file per data source ("data system"):
                       fetch_vulekamali.R, fetch_igfr.R, fetch_statssa_cod.R,
                       fetch_ncodv.R, fetch_gbd.R, fetch_dhb.R, fetch_ghs.R,
                       fetch_sadhs.R, plus crosswalk_programmes.R and
                       extract_app_citations.R (strategic-document citation
                       trail)
                     shared: constants.R, utils_errors.R
scripts/             one-off runnable scripts, e.g. run_first_implementer.R
                       (real end-to-end budget + citation-trail pull)
data/raw/            untouched pulls, one subfolder per source, never
                       hand-edited (gitignored — refetched via the pipeline;
                       manual-download sources are placed here by hand)
data/processed/      harmonised/joined outputs
data/crosswalks/     programme-structure mapping + naming inconsistencies
analysis/            Quarto documents for ad hoc exploratory work (not the
                       published site — see site/ and "Exploring the
                       project" below)
site/                Quarto website project (chapters, landing page,
                       shared R/load_data.R) — the project's explorable,
                       publishable multi-chapter dashboard
docs/
  KNOWLEDGE.md        the "what we know" register
  PLAN.md             the "what we plan to measure" register
tests/testthat/       testthat coverage per fetcher (non-empty result,
                       expected columns, expected provinces/years present)
_targets.R            pipeline definition
```

## Setup

```r
renv::restore()   # installs the locked package versions from renv.lock
```

`tabulapdf` (the ropensci PDF-table-extraction package, formerly named
`tabulizer`) needs a working `rJava`, which needs R built with Java support.
If `renv::restore()` fails on `rJava`, run `R CMD javareconf` first (see
`docs/KNOWLEDGE.md` "Environment notes").

## Running the pipeline

```r
targets::tar_manifest()      # inspect the target graph without running it
targets::tar_make()          # run the pipeline (expected to fail loudly on
                              # unimplemented fetchers until they're built out)
```

To run just the two real, working implementers end-to-end without hitting
the still-stubbed fetchers:

```sh
Rscript scripts/run_first_implementer.R
```

## Running tests

```sh
Rscript tests/testthat.R
```

## Exploring the project

`site/` is a Quarto **website** project organised into chapters (one `.qmd`
file per chapter under `site/chapters/`, listed as cards on `site/index.qmd`):
Overview, Budget Panel, Programme Crosswalk & Data Quality, Citation Trail,
and Plan & Open Questions. It reads directly from `data/processed/` and
`data/crosswalks/` via `site/R/load_data.R`, so run
`scripts/run_first_implementer.R` at least once first. Render and preview
with:

```sh
cd site
quarto render      # writes site/_site/ (gitignored)
quarto preview      # live-reload local preview
```

A single-file `format: dashboard` version was tried first but dropped in
favour of this website structure: dashboard format's viewport-tiled rows
fight long-form chapter content, and it has no landing-page listing. The
website keeps the same interactive filtering (crosstalk + DT + plotly) but
as normal scrolling pages, one per chapter.

Requires the `DT`, `ggplot2`, `plotly`, `crosstalk`, and `scales` packages
(installed via `renv::restore()`).

### Publishing to GitHub Pages

Create a **separate, website-only GitHub repository**. Keep this research
folder local: do not push it, its data, or its Git history to that repository.
The deployment script renders the site locally, then pushes only the rendered
website assets to the remote's dedicated `gh-pages` branch. It does not require
this research folder to be a Git repository.

```sh
# First, render and inspect the publication file list without uploading:
bash scripts/deploy_site.sh --dry-run

# Publish (replace OWNER/REPOSITORY with the repository you created):
bash scripts/deploy_site.sh --repo git@github.com:OWNER/REPOSITORY.git
```

HTTPS Git URLs also work with your configured Git credentials. You need Quarto,
the project's R packages and locally generated data, Git, rsync, a configured
Git author (`user.name`/`user.email`), and push access to the remote. No data
pull is run automatically; update your local data first when needed.

After the first deployment, open the GitHub repository's **Settings > Pages**:
choose **Deploy from a branch**, select **gh-pages** and **/ (root)**, then
save. GitHub displays the published URL there (usually
`https://OWNER.github.io/REPOSITORY/`). Run the same deploy command whenever
you want to re-render and update the website.

The `gh-pages` branch is managed entirely by the script: files not in the new
render are removed, so do not store other material there. Other remote branches
are not modified, and updates are never force-pushed. A failed render or
authentication/push error stops the deployment and reports failure.

Only HTML and required CSS, JavaScript, images, fonts, Quarto's search index,
and `.nojekyll` are published. Raw/processed dataset files, PDFs, R/Quarto
source, internal registers, caches, and dependency lockfiles are not copied.
**Interactive charts and tables embed their displayed values in HTML; those
values are public and extractable.** This is not a way to conceal values
shown on the site. Links to original public sources are retained.

Deployment regression checks use a fixture renderer and a temporary local Git
remote (no GitHub access or research data required):

```sh
bash tests/test-deploy-site.sh
```
# Policy_Landscape
