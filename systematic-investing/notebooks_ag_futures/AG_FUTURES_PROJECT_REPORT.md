# Agricultural Futures Final Project Report

## Overview

This project studies whether a systematic crop-futures strategy can achieve a higher Sharpe ratio by combining standard technical trading rules with information about crop health. The project is centered on corn futures, uses wheat futures for a relative-value pairs sleeve, and uses NASA vegetation data as a proxy for crop-health conditions.

The final research question is:

> Can a diversified portfolio of raw crop-futures trading strategies be improved by forward-looking crop-health information, and if so, at what realistic forecast horizon does that information become most valuable?

The project is implemented in four notebooks:

- [ag_futures_final_project.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/ag_futures_final_project.ipynb)
- [corn_futures_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/corn_futures_data_builder.ipynb)
- [wheat_futures_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/wheat_futures_data_builder.ipynb)
- [mod13a2_ndvi_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/mod13a2_ndvi_data_builder.ipynb)

The builder notebooks isolate data ingestion and cleaning. The main notebook focuses on strategy construction, evaluation, portfolio selection, and interpretation.

## Motivation

The motivation is straightforward: build a crop-futures trading process with a high Sharpe ratio.

Pure technical trading rules can work in agricultural markets, but they ignore the most important fundamental driver of crop supply: weather and resulting crop health. If crop conditions can be anticipated before they are fully reflected in futures prices, then weather-related information may help either:

- improve directional timing in corn futures, or
- improve the quality of a diversified portfolio of raw trading sleeves.

At the same time, crop-health data must be used carefully. Raw vegetation levels are highly seasonal. A low winter NDVI reading does not mean corn is unusually weak; it mostly means winter looks like winter. For that reason, the project moved away from raw NDVI levels and toward seasonally adjusted crop-health anomalies.

## Hypothesis

The project tests four related hypotheses:

1. A diversified combination of raw technical crop-futures strategies should outperform most standalone sleeves on a Sharpe basis.
2. Raw NDVI level is not an economically sensible trading signal, but seasonally adjusted NDVI anomaly may be.
3. Crop-health information is more useful as a regime or overlay than as a standalone rule tied to a single technical strategy.
4. A realistic medium-term weather-forecast horizon may be more valuable than either a very short horizon or a very long one.

## Data

The final notebook uses three prepared datasets:

- Corn futures: `6,452` daily rows from `2000-07-17` to `2026-04-30`
- Wheat futures: `6,464` daily rows from `2000-07-17` to `2026-04-30`
- NDVI daily panel: `9,546` daily rows from `2000-02-18` to `2026-04-07`

The corn-NDVI overlap is:

- `6,435` daily rows from `2000-07-17` to `2026-04-07`

The final raw-strategy optimization sample is slightly shorter because every strategy needs enough lookback history:

- `6,159` daily rows from `2001-10-02` to `2026-04-07`

In the report and notebook, this shorter window is called the **common NDVI-overlap sample**. That means the single shared date range where:

- the NDVI series exists,
- the relevant futures series exist, and
- every raw strategy has enough lookback history to produce returns.

Using that common sample makes all strategy comparisons apples-to-apples.

### Futures data

Corn and wheat are stored as continuous, back-adjusted futures series. The project uses those adjusted close series so that standard backtests are not distorted by contract-roll jumps.

### NASA data

The NASA input is MODIS `MOD13A2` vegetation-index data. The raw archive contains `602` composite observations from `2000-02-18` to `2026-04-07`.

The data-builder notebook processes the files one at a time:

1. open one HDF file
2. extract NDVI
3. remove invalid and fill values
4. compute a regional mean
5. store only `date + regional mean NDVI`

This design avoids loading the entire raw archive into memory.

## NASA Signal Construction

The final project does **not** use raw NDVI level directly as the economic signal.

Instead, the workflow is:

1. reduce each MODIS composite file to a regional mean NDVI
2. place the 16-day composite observations on a daily index by forward-filling between releases
3. compute a seasonal benchmark for each calendar day using the full historical sample
4. calculate the deviation of current NDVI from that seasonal norm
5. standardize that deviation into a z-score

Formally:

- `seasonal_mean(t)` = average NDVI for that same calendar point across years
- `ndvi_anomaly(t)` = `ndvi(t) - seasonal_mean(t)`
- `ndvi_anomaly_z(t)` = `ndvi_anomaly(t) / seasonal_std(t)`

This is better described as **seasonal adjustment** than smoothing. The point is to answer:

> Is crop health unusually weak or strong for this point in the crop calendar?

The crop-health regime is only activated in the active growing window:

- April through October

This prevents the model from making the naive inference that low winter vegetation should suppress trading.

## Weather-Forecast Tiers

The project uses three lead windows as stylized forecast-skill tiers:

- `basic`: `14` days
- `advanced`: `45` days
- `expert`: `90` days

These are not literal weather forecasts. They are **oracle-style benchmarks** using future realized NDVI anomaly as a stand-in for what increasingly strong weather-model skill might approximate.

The interpretation is:

- `basic`: information roughly consistent with ordinary short-range/subseasonal forecasting
- `advanced`: strong sub-seasonal forecasting ability
- `expert`: unusually strong seasonal foresight

This creates a clean theoretical exercise:

> If a hedge fund could forecast future crop-health anomalies at different realistic horizons, how much would that improve a systematic crop-futures portfolio?

## Strategy Set

The raw strategy universe contains:

### Trend

- moving-average `10/30`
- moving-average `30/100`
- moving-average `80/160`

### Counter-trend

- a buy-the-dip rule based on prior highs and average trading range with `p = 2.2`

### Volatility regime

- a rule that switches exposure based on the recent volatility state of corn futures

### Pairs

- corn-wheat relative-value strategies using `5-day`, `10-day`, and `20-day` spread horizons

The pairs section first checks whether the corn-wheat spread appears stationary enough to justify a mean-reversion test. The Augmented Dickey-Fuller result was supportive:

- `ADF statistic = -3.9752`
- `p-value = 0.0015`

That does not prove profitability, but it does justify testing the spread-trading idea.

## Standalone Strategy Results

On the common optimization sample, the strongest raw standalone sleeves were:

- `vol_regime`: Sharpe `0.4671`
- `ma_10_30`: Sharpe `0.3698`
- `ma_80_160`: Sharpe `0.1728`
- `pairs_10d`: Sharpe `0.0849`

Weak or losing raw sleeves included:

- `countertrend_p2_2`: Sharpe `-0.0183`
- `ma_30_100`: Sharpe `-0.0985`
- `pairs_20d`: Sharpe `-0.1825`

This already says something important: the classic technical sleeves are not equally effective in this market, and some course-style rules are much more useful as ingredients than as final standalone strategies.

## Why Portfolio Construction Was Necessary

The notebook includes a correlation matrix of raw strategy returns on the common NDVI-overlap sample, meaning the single shared evaluation window where NDVI, futures data, and all raw strategy return series are simultaneously available. The purpose of that chart is to test whether diversification is real.

The main conclusion from the correlation structure is:

- outright corn sleeves are not perfectly redundant
- the pairs sleeve is largely separate from the outright corn sleeves
- some strategies that are only modestly attractive alone can still help portfolio Sharpe through low correlation

That is why the project does not simply pick the single best strategy. It explicitly searches for the best **combination** of raw sleeves.

## Raw Strategy Combination

The portfolio search considers every non-empty subset of the raw strategy universe. For each subset, it chooses **long-only weights** that maximize annualized Sharpe on the common sample.

This is an important design decision. The project allows a weak standalone sleeve to survive **if** it improves the total portfolio through diversification. However, it does not permit shorting weak strategies into synthetic new ones. That keeps the exercise academically cleaner and easier to explain.

### Selected raw combination

The best raw combination used four sleeves:

- `vol_regime`: `36.71%`
- `ma_10_30`: `28.09%`
- `pairs_10d`: `18.37%`
- `ma_80_160`: `16.83%`

Performance of the selected raw combination:

- annualized Sharpe: `0.6343`
- cumulative return: `6.9749`
- annualized volatility: `0.1524`
- max drawdown: `-0.3605`

This is materially better than the best raw standalone sleeve:

- best raw standalone: `vol_regime`, Sharpe `0.4671`
- best raw combination: Sharpe `0.6343`

That is one of the central discoveries of the project: **diversification across raw strategy sleeves matters a lot.**

Interestingly, the final optimum did **not** need a negative-Sharpe raw sleeve once correlation structure was taken into account. The optimizer still considered them, but the best solution concentrated in four sleeves with stronger standalone and diversification characteristics.

## Applying Weather Information to the Final Combination

After selecting the raw portfolio, the project keeps those raw weights fixed and then applies the crop-health overlay.

This is the cleanest academic design because it separates two effects:

1. the value of combining raw systematic strategies
2. the value of weather or crop-health foresight

If the weights were re-optimized after each NDVI overlay, the attribution would become much less clear.

### Results

The fixed-weight portfolio results were:

- `combo_raw_optimal`: Sharpe `0.6343`
- `combo_raw_optimal_ndvi_basic`: Sharpe `0.6245`
- `combo_raw_optimal_ndvi_advanced`: Sharpe `0.7829`
- `combo_raw_optimal_ndvi_expert`: Sharpe `0.6225`

Sharpe uplift versus the raw combination:

- `basic`: `-0.0098`
- `advanced`: `+0.1486`
- `expert`: `-0.0118`

The result is surprisingly sharp:

> Only the `advanced` lead window improved the final diversified portfolio.

## Interpretation of the Final Result

This is the most important conclusion of the project.

The final results suggest that:

- very short-horizon crop-health foresight is not enough to materially improve the diversified portfolio
- extremely long-horizon foresight may be too diffuse, too noisy, or too early to align with trading signals
- a medium-horizon, sub-seasonal forecast window appears to be the most valuable

In economic terms, the project points toward the following interpretation:

> If a trading operation had strong weather-model skill, the most useful edge would likely come from anticipating crop-health conditions several weeks ahead, not just a few days ahead and not a full season ahead.

That is a much more realistic and interesting result than “perfect weather foresight makes money.” It suggests there may be a specific horizon at which weather information becomes actionable for systematic crop-futures trading.

## Additional Findings

Several secondary findings also matter:

- The original raw NDVI level was not a sensible final signal.
- Seasonal adjustment was necessary to make the NASA data economically interpretable.
- The volatility-regime sleeve was the strongest raw standalone rule.
- The `10/30` moving-average rule was much stronger than the `30/100` baseline.
- The corn-wheat spread passed a basic stationarity test, so the pairs idea was worth exploring.
- NDVI-led pairs variants did improve some pairs sleeves, but the strongest final result came from applying weather information to the optimized raw portfolio rather than treating pairs as the headline strategy.

## Limitations

This project is still a simplified academic exercise, and the main limitations should be stated clearly:

- The weather overlay uses future realized NDVI anomaly as an oracle benchmark, not a tradable forecast.
- No transaction costs, slippage, funding costs, or margin constraints are modeled.
- The crop-health proxy is NDVI only; it does not yet include soil moisture, land-surface temperature, or precipitation.
- The geographic aggregation is simplified into a regional NDVI signal rather than a more granular agronomic model.
- The results are in-sample and should be interpreted as exploratory, not production-ready.

These limitations do not invalidate the exercise. They define what the exercise is:

> a structured test of whether weather-model-style crop-health foresight is worth adding to systematic futures trading.

## Final Conclusion

The project reached three main conclusions:

1. A diversified combination of raw crop-futures strategies outperformed the individual raw sleeves on a Sharpe basis.
2. NASA vegetation data became economically meaningful only after seasonal adjustment and crop-calendar filtering.
3. The most valuable weather-information tier was the `advanced` lead horizon, which improved the optimized raw portfolio Sharpe from `0.6343` to `0.7829`.

The final interpretation is that the best use of crop-health information is **not** as a crude standalone NDVI trade. Instead, it is best viewed as a **forward-looking regime overlay** on top of an already-diversified systematic crop-futures portfolio.

That makes the final project both academically coherent and economically intuitive:

- first build a good systematic trading engine
- then ask whether realistic forecast skill in weather and crop health makes that engine better

In this notebook, the answer is yes, but only at the right lead horizon.

## Reproducibility

The intended workflow is:

1. run [corn_futures_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/corn_futures_data_builder.ipynb)
2. run [wheat_futures_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/wheat_futures_data_builder.ipynb)
3. run [mod13a2_ndvi_data_builder.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/mod13a2_ndvi_data_builder.ipynb)
4. run [ag_futures_final_project.ipynb](/Users/jlaw/projects/stern/systematic-investing/notebooks_ag_futures/ag_futures_final_project.ipynb)

The cached CSV inputs live in [data_ag_futures](/Users/jlaw/projects/stern/systematic-investing/data_ag_futures), while the raw NASA archive can be provided separately and unzipped into:

- [data_ag_futures/raw/nasa/mod13a2](/Users/jlaw/projects/stern/systematic-investing/data_ag_futures/raw/nasa/mod13a2)
