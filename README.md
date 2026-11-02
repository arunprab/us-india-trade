# US–India Goods Trade, 1995–2024

**Author:** Arun Kumar Prabhakaran
**Date:** 2026-11-01

A data project on what the United States and India sell to each other and how that mix has changed since 1995. It pulls goods-trade data from the UN Comtrade public API, cleans and validates it, analyzes it (totals and balance, composition and shifts, concentration, the gap between the two countries' own reports, drivers of growth, product detail) and visualizes the results. Everything is in one Jupyter notebook: `us_india_trade_analysis.ipynb`.

This is a sibling of the [`wiki-web-scraping`](https://github.com/arunprab/wiki-web-scraping) project and follows the same approach (collect, clean, analyze, visualize, conclude), with a different source.

## What it covers

- **Collect:** one API call per country, direction and year for the 97 Harmonized System (HS) chapters, 1995–2024, plus 4-digit product detail for the ten biggest chapters in four benchmark years. Every response is cached on disk.
- **Validate:** chapter sums are checked against Comtrade's own totals, and missing years are reported.
- **Analyze:** totals and balance, top commodities and share shifts, concentration (top-5 share, HHI), US vs India reporting ("mirror" data), growth contributors, top products.
- **Limits to keep in mind:** goods only (no services), current US dollars (not inflation-adjusted), and the Harmonized System is revised periodically, so 4-digit codes are less comparable over time than chapters.

## Project layout

```
us_india_trade_analysis.ipynb   the notebook (start here)
requirements.txt                Python dependencies
figures/                        the charts as PNG files (not in git; created by the run)
data/raw/comtrade/              cached API responses (not in git; created by the first run)
data/clean/                     cleaned CSVs (not in git; created by the run)
```

## Requirements

- Python 3.9 or newer
- An internet connection on the first run. The first run makes about 150 rate-limited API calls and takes several minutes (longer if the API quota has to reset); later runs read the cache and take seconds.

## Setup

Run these from the project folder.

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**Windows (PowerShell)**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Running the notebook

Start from the project folder: the notebook reads and writes `data/` and `figures/` relative to the folder it is opened from. If you activated the virtual environment, you can drop the `.venv/bin/` prefix.

- **Jupyter Notebook:** `.venv/bin/jupyter notebook us_india_trade_analysis.ipynb`, then **Kernel → Restart & Run All**.
- **JupyterLab:** `.venv/bin/jupyter lab us_india_trade_analysis.ipynb`, then **Run → Run All Cells**.
- **VS Code:** open the folder, open the notebook, pick the `.venv` interpreter as the kernel, then **Run All**.
- **Without opening anything:**
  ```bash
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 us_india_trade_analysis.ipynb
  ```
  The `timeout=-1` matters on the first run, because the default per-cell time limit is shorter than the download takes.

## Data, caching and refreshing

- The notebook uses Comtrade's **public preview API, which needs no key**. It allows one year per request and at most 500 rows per response, is rate-limited, and returns codes without descriptions. The code works around all of that: it makes one request per year, asks for explicit product-code lists instead of "all products", waits between calls, retries politely on HTTP 429, and takes commodity names from Comtrade's public HS reference file.
- Responses are cached in `data/raw/comtrade/` and never expire on their own, so re-running later reuses the old snapshot. To pull fresh data, set `REFRESH = True` in the notebook's first code cell and run all cells (every response is re-downloaded once and replaces its cached copy; a failed download keeps the old copy and stops with an error), then set it back to `False`. Deleting the cache folder has the same effect.
- A refresh overwrites `data/clean/` and `figures/`. The written conclusions at the end of the notebook are static text from the original snapshot, so review them after refreshing. Recent years may also be revised by the reporting countries.

## Data source and terms

Data: **UN Comtrade** (United Nations Statistics Division), downloaded through its public API. Cite it as "UN Comtrade Database, comtradeplus.un.org" with the access date.

UN Comtrade's terms restrict redistribution and commercial use of its data, and require attribution. For that reason **the downloaded data and the cleaned CSVs are not committed to this repository** (`data/` is git-ignored), the notebook is committed **without its outputs**, and the generated charts in `figures/` are not committed either. Run the notebook to download the data and generate the tables and charts yourself. Read Comtrade's current terms before reusing the data, or anything built from it, beyond personal or educational use.

## License

The code and notebook are released under the [MIT License](LICENSE). The MIT license covers the code only; the underlying data is UN Comtrade's and keeps its own terms, as described above.

## Troubleshooting

**HTTP 429 or "rate limited, waiting …" messages**
Normal on the first run. The notebook waits and retries automatically.

**"API call quota used up, waiting N min for it to reset"**
Comtrade's public endpoint has a call-volume quota (on the order of a hundred calls per hour). When it runs out the notebook reads the reset time from the API's message, waits, and carries on; every response already downloaded is cached, so no work is repeated. If the reset is more than an hour away it stops with a clear message; just run it again later. A full first run may therefore take a while, but later runs need no API calls at all.

**The notebook stops with a timeout under `nbconvert`**
Use `--ExecutePreprocessor.timeout=-1` as shown above, or run it from Jupyter, which has no per-cell limit.

**"WARNING: … hit the 500-row cap"**
A response was truncated by the API's row limit. The code only asks for amounts that fit, so this should not appear; if it does, results for that query are incomplete.

**The kernel can't find pandas / "No module named …"**
The notebook is running with a different Python than the one you installed the requirements into. Re-select the `.venv` kernel, or start Jupyter with `.venv/bin/jupyter`.
