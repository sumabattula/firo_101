# FIRO 101

A HydroLearn module notebook that walks through a forecast-informed reservoir
operations (FIRO) analysis for Lake Mendocino (station `LAMC1`), using CNRFC
HEFS ensemble forecasts and CDEC observed storage/outflow data.

**Authors:** Sujana Timilsina, Suma Bhanu Battula

## What's in this repo

| File | Purpose |
|---|---|
| `FIRO_module_Hydrolearn.ipynb` | The main notebook: downloads a HEFS ensemble forecast, ranks members by volume, picks representative traces, projects storage against the guide curve, and compares to actual observed outflow. |
| `COY_15_2025.xlsx` | Cached observed **storage** data (CDEC sensor 15) for water year 2025. |
| `COY_23_2025.xlsx` | Cached observed **outflow** data (CDEC sensor 23) for water year 2025. |
| `requirements.txt` | Python dependencies. |
| `.github/workflows/run-notebook.yml` | GitHub Actions workflow that executes the notebook on push/PR and uploads the run as an artifact. |

## Quick start (local)

```bash
git clone https://github.com/sumabattula/firo_101.git
cd firo_101
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook FIRO_module_Hydrolearn.ipynb
```

Open the notebook and run all cells. The only cell you should normally need
to edit is the **User Inputs** cell, where you can set:

- `station` — CNRFC station ID (default `LAMC1` = Lake Mendocino)
- `issue_date`, `cycle`, `group`, `kind` — which HEFS forecast issuance to pull
- `analysis_start`, `analysis_end` — the sub-window of the forecast to analyze
- `target_percentiles` — which representative ensemble traces to select (e.g. P05/P50/P95)

## Running it on GitHub (Actions)

The included workflow (`.github/workflows/run-notebook.yml`) will:

1. Check out the repo
2. Install Python and the packages in `requirements.txt`
3. Execute the notebook with `nbconvert`
4. Upload the executed notebook (with all outputs/plots filled in) as a
   downloadable build artifact

It runs automatically on every push/PR that touches a `.ipynb` file, or you
can trigger it manually from the **Actions** tab → *Run FIRO Notebook* →
*Run workflow*.

## ⚠️ Live data dependency

This notebook **downloads data at runtime** rather than only reading local
files:

- The HEFS ensemble forecast is pulled from `cnrfc.noaa.gov` based on the
  `issue_date`/`cycle`/`group`/`kind` set in the User Inputs cell. CNRFC does
  not keep every past issuance available indefinitely, so an old
  `issue_date` may eventually 404.
- Observed storage/outflow are pulled fresh from `cdec.water.ca.gov` for the
  configured `water_year` (set `CONFIG["download_observed_data"] = False` to
  skip this and use the local `COY_15_2025.xlsx` / `COY_23_2025.xlsx` files
  instead).
- If the CDEC download fails, the notebook falls back to fetching cached
  copies of those two files from GitHub — **but that fallback currently
  points at a different repo** (`suzanatimilsina/Hydrolearn_FIRO`), not this
  one. Update the `GITHUB_BASE` URL in the download-fallback cell if you want
  it to pull from this repo's own cached copies instead.

Because of this, a CI run can fail or be slow for reasons unrelated to the
notebook's code — e.g. an unreachable host, a rate limit, or an aged-off HEFS
issuance. If you need fully reproducible CI runs regardless of network
conditions, set `download_observed_data = False` and point `hefs_file` at a
committed, already-extracted CSV instead of triggering a fresh download.

## Outputs

Running the notebook creates (or overwrites) an `outputs/` folder containing:

- Volume ranking, selected-traces, and storage-rise CSVs
- Additional-outflow-needed CSVs and summary
- Actual-vs-required outflow comparison CSV and summary
- PNG figures for each of the above analyses

## License

No license specified yet — add one (e.g. MIT) if you intend for others to
reuse this module.
