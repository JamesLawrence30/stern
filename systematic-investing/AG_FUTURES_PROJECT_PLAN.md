# Corn Futures + MODIS NDVI Project Plan

## Project Thesis
- Build the final project around one clear claim: `MOD13A2` vegetation stress can improve a course-style corn futures trend rule when used as a slower regime filter rather than as a standalone daily predictor.
- Keep the MVP narrow and defensible:
  - Market: Corn futures only
  - NASA signal: MODIS NDVI only
  - Technical baseline: moving-average crossovers learned in class, with `30/100` as the primary overlay rule
- Use the overlapping sample between corn futures history and the NDVI file coverage actually available on disk.

## Notebook Build Order
1. Inspect the local NDVI inventory and summarize the file-date coverage from the MOD13A2 filenames.
2. Download corn futures from `ZC=F` and create an academic back-adjusted continuous series using the roll-month schedule `H/K/N/U/Z`.
3. Evaluate the class-style baseline rules `10/30`, `30/100`, and `80/160`.
4. If the NDVI inventory is long enough and readable, parse the HDF files, compute a mean NDVI series, and convert it into a daily regime signal.
5. Apply the NDVI regime overlay to the `30/100` corn rule:
   - full size when technical and NDVI agree
   - half size when NDVI is neutral
   - flat when NDVI conflicts
6. Compare buy-and-hold, raw technical, and NDVI-filtered results in one summary table and a few plots.

## Student Tasks
- Re-download a longer MOD13A2 history if needed. The notebook now checks the local inventory dynamically.
  - this note is stale once new files are added; the notebook now computes the actual coverage dynamically
- Target a true 16-day time series, not annual spot checks. The clean target remains `2016-01-01` onward.
- Decide whether to keep the raw NASA files in `/Users/jlaw/projects/earthdata_downloads` or move/copy them into `/Users/jlaw/projects/stern/systematic-investing/data/ag_futures/raw/nasa/mod13q1`.
- If you re-download, prefer a workflow that is easier to parse:
  - best option: AppEEARS subset in NetCDF
  - workable option: keep HDF4 and install an HDF4 reader such as `pyhdf`
- After the longer NDVI pull is ready, rerun the notebook so the overlay section can activate instead of skipping.

## AI Tasks
- Replace the notebook stub in [ag_futures_final_project.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks/ag_futures_final_project.ipynb) with a structured corn-only workflow.
- Build the corn back-adjustment logic around the class-appropriate roll approximation for `ZC=F`.
- Implement the baseline moving-average backtests and save processed outputs into `data/ag_futures/processed/` when the notebook is run.
- Inspect the NDVI inventory from the local Earthdata folder and explicitly block the overlay when coverage is too short.
- Add a MOD13A2 HDF4 reader hook that uses `pyhdf` when available, with a clear error if the parser dependency is missing.
- Keep the main notebook on a CSV-first workflow with explicit refresh flags instead of unconditional live downloads.
- Use a separate builder notebook for any future Yahoo probing or corn-cache regeneration: [corn_futures_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks/corn_futures_data_builder.ipynb).

## Data Folder Map
- `data/ag_futures/raw/futures/`
  - reserved for any manually saved corn futures extracts if we decide to persist raw downloads locally
- `data/ag_futures/raw/nasa/mod13q1/`
  - intended repo-local home for MOD13A2 raw files
- `data/ag_futures/processed/`
  - notebook outputs such as:
  - `corn_continuous_back_adjusted.csv`
  - `corn_ndvi_level_series.csv`
  - `corn_ndvi_daily_series.csv`
  - `corn_ndvi_overlay_panel.csv`
- External Earthdata folder currently detected:
  - `/Users/jlaw/projects/earthdata_downloads`

## Done / Blocked
- Done:
  - created the repo-side project tracker
  - created the local `ag_futures` data scaffold
  - narrowed the notebook MVP to `corn + NDVI only`
  - added explicit logic for the current Earthdata inventory check
  - added `REFRESH_CORN_FROM_YAHOO = False` to keep the project notebook on a stable local-cache path
  - split the corn-history retrieval workflow into a separate builder notebook
- Blocked:
  - the local MOD13A2 inventory still needs to look like a real 16-day series before the NDVI overlay is credible
  - the notebook now checks both total coverage and median gap spacing so it can distinguish real time-series pulls from annual snapshots
