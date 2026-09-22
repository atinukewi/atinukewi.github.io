# atinukewi.github.io

This is the source for my personal website and blog — an "About" page and a set of
milestone posts (mixing R and Python analysis) built with [Quarto](https://quarto.org)
and published to GitHub Pages.

## 1. Install first

You need three things on your machine before you touch this repo. Versions are what
this project was built and last rendered with; close versions should work, but these
are the known-good ones.

| Tool   | Version used | Notes |
|--------|---------------|-------|
| Quarto | 1.10.18       | [quarto.org/docs/get-started](https://quarto.org/docs/get-started/) |
| `uv`   | 0.11.17       | Manages the Python interpreter *and* the Python deps — you do not need a separate Python install. `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| R      | 4.6.1         | [r-project.org](https://www.r-project.org/) — needs to be on your `PATH` as `R` / `Rscript` |

`renv` (the R package manager) is **not** something you install yourself — it bootstraps
itself the first time R runs inside this project (see step 2 below).

## 2. Build, start to finish

Run the shell commands from your terminal, in order. The `Rscript -e` command runs R
code but is still typed at the shell — R itself only opens implicitly to run that one
line.

```bash
# 1. Clone the repo (shell)
git clone https://github.com/atinukewi/atinukewi.github.io.git
cd atinukewi.github.io

# 2. Set up the Python environment (shell)
# Reads pyproject.toml / uv.lock, downloads Python 3.14 if you don't have it,
# and installs pandas, seaborn, palmerpenguins, jupyter, etc. into .venv/
uv sync

# 3. Set up the R environment (shell, runs a one-line R script)
# The first R process started in this folder sources .Rprofile, which runs
# renv/activate.R. That script installs renv itself if it's missing, then
# renv::restore() reads renv.lock and installs the exact R package versions
# (tidyverse, palmerpenguins, etc.) into a project-local library.
Rscript -e "renv::restore()"

# 4. Render the site (shell)
# Runs inside the uv-managed venv so the Python code blocks (jupyter: python3)
# pick up the packages from step 2. The R code blocks use Rscript from your
# PATH, which auto-activates the renv library from step 3.
uv run quarto render
```

If `renv::restore()` prompts you to confirm, answer yes — it's just agreeing to
install the locked package versions into the project library.

## 3. Where the built site lands

`_quarto.yml` sets `output-dir: docs`, so every render writes the static site into the
`docs/` folder at the repo root (that folder is also what GitHub Pages serves the live
site from).

To view it locally, either:

```bash
# Live-reloading preview server (rebuilds as you edit .qmd files)
quarto preview
```

or, if you just want to look at the already-rendered output without a rebuild:

```bash
# Any static file server pointed at docs/
cd docs
python -m http.server 8000
# then open http://localhost:8000
```

Opening `docs/index.html` directly as a `file://` URL mostly works too, but a local
server avoids relative-path/asset-loading quirks.

## 4. Where the data comes from

All the analysis posts use the **Palmer Penguins** dataset (Horst, Hill & Gorman 2020,
Palmer Station Antarctica LTER, doi: 10.5281/zenodo.3960218), via the `palmerpenguins`
package — the R version (`library(palmerpenguins)`) and the Python version
(`from palmerpenguins import load_penguins`).

Both packages ship the dataset as a local file bundled inside the package itself. That
means:

- **Installing** the packages (`uv sync` / `renv::restore()`) needs network access, to
  download the packages from PyPI / CRAN.
- **Rendering** the site (`quarto render`) does **not** need network access — `load_penguins()`
  and `penguins` read the data straight from disk, with no HTTP calls at build time.
