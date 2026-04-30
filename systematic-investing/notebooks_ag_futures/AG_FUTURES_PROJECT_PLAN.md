# Corn Futures + MODIS NDVI Project Plan

## Project Thesis
- Build the final project around one clear claim: `MOD13A2` vegetation stress can improve a course-style corn futures trend rule when used as a slower regime filter rather than as a standalone daily predictor.
- Keep the current workflow modular:
  - Data builders prepare clean CSV caches for corn, wheat, and MODIS NDVI
  - The final project notebook only loads those caches and evaluates strategies
  - Technical baseline: moving-average crossovers learned in class, with `30/100` as the primary NDVI overlay rule
- Use the overlapping sample between the prepared futures caches and the prepared NDVI regime series.

## Notebook Build Order
1. Build or refresh the corn futures cache in [corn_futures_data_builder.ipynb](./corn_futures_data_builder.ipynb).
2. Build or refresh the wheat futures cache in [wheat_futures_data_builder.ipynb](./wheat_futures_data_builder.ipynb) if pairs analysis is desired.
3. Build or refresh the NDVI level and daily regime CSVs in [mod13a2_ndvi_data_builder.ipynb](./mod13a2_ndvi_data_builder.ipynb).
4. In the main project notebook, load the prepared CSVs and evaluate the class-style baseline rules `10/30`, `30/100`, and `80/160`.
5. Apply the NDVI regime overlay to the `30/100` corn rule.
6. Compare trend, counter-trend, volatility-regime, optional pairs, and weighted combinations in one summary table and a few plots.

## Student Tasks
- Re-download a longer MOD13A2 history if needed. The notebook now checks the local inventory dynamically.
  - this note is stale once new files are added; the notebook now computes the actual coverage dynamically
- Target a true 16-day time series, not annual spot checks. The clean target remains `2016-01-01` onward.
- Keep the raw NASA files in `/Users/jlaw/projects/earthdata_downloads` unless you want a repo-local mirror later.
- If you re-download, prefer a workflow that is easier to parse:
  - best option: AppEEARS subset in NetCDF
  - workable option: keep HDF4 and install an HDF4 reader such as `pyhdf`
- After data-builder notebooks are refreshed, rerun the main project notebook from the top so all strategy comparisons use the newest caches.

## AI Tasks
- Keep data cleaning and aggregation in standalone builder notebooks.
- Use [corn_futures_data_builder.ipynb](./corn_futures_data_builder.ipynb) for corn history.
- Use [wheat_futures_data_builder.ipynb](./wheat_futures_data_builder.ipynb) for wheat history.
- Use [mod13a2_ndvi_data_builder.ipynb](./mod13a2_ndvi_data_builder.ipynb) for NDVI extraction, cleaning, and regime caching.
- Keep [ag_futures_final_project.ipynb](./ag_futures_final_project.ipynb) focused on strategy construction, Sharpe comparisons, and signal combination.

## Data Folder Map
- `data_ag_futures/raw/futures/`
  - `zc_continuous_chunked_raw.csv`
  - `zw_continuous_chunked_raw.csv`
- `data_ag_futures/processed/`
  - tracked core caches:
  - `corn_continuous_back_adjusted.csv`
  - `wheat_continuous_back_adjusted.csv`
  - `mod13a2_ndvi_level_series.csv`
  - `mod13a2_ndvi_daily_series.csv`
- External Earthdata folder currently detected:
  - `/Users/jlaw/projects/earthdata_downloads`

## Done / Blocked
- Done:
  - created separate builder notebooks for corn, wheat, and NDVI preprocessing
  - moved the main notebook onto a strategy-only, CSV-first workflow
  - cached cleaned corn, wheat, and NDVI series under `data_ag_futures/processed/`
- Blocked:
  - none for the current MVP; remaining work is strategy refinement rather than data plumbing
