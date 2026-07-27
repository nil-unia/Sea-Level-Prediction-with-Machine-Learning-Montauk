# Analysis Brief: Forecasting Sea Level Rise with Prophet

## 1. Objective

Sea level rise is one of the more measurable, data-rich consequences of climate change, which makes it a strong candidate for applying time series machine learning. It has a clear real signal, the historical record is long (NOAA tide gauge data goes back over a century at many stations), and the results are directly relevant to public infrastructure and community planning.

The goal of this project was to:
1. Build a reproducible pipeline that takes raw NOAA monthly sea level records and turns them into a trained forecasting model.
2. Quantify how well that model performs on historical hold-out data before trusting it with long-range projections.
3. Compare the model's own long-range forecast against NOAA's published regional sea level rise scenarios, which are grounded in physical climate models rather than pure statistical extrapolation.

## 2. Data & Methods

**Source:** NOAA Sea Level Trends station data (monthly mean sea level, `Monthly_MSL`, in meters relative to station datum).

**Station used:** Montauk, NY : monthly records from 1947 through 2026, giving ~79 years of history.

**Preprocessing:**
- Combined the `Month` and `Year` integer columns into a proper datetime (`ds`) column, as required by Prophet's input format.
- Set `ds` as the DataFrame index and dropped the now-redundant `Month`/`Year` columns.

**Train/test split:** Rather than a random split (which would leak future information into training for a time series), the data was split by date:
- **Training set:** 1975-01-01 → 2000-01-01 (25 years)
- **Test set:** 2000-01-01 → 2026-01-01 (26 years, held out)

**Model:** Meta's Prophet, fit with default parameters (additive trend + yearly seasonality; weekly/daily seasonality auto-disabled since the data is monthly).

**Evaluation metrics:**
- **Mean Absolute Error (MAE)** : average absolute deviation between forecast and actual, in the same units as the data (meters).
- **Symmetric Mean Absolute Percentage Error (SMAPE)** : a percentage-based error metric that treats over- and under-prediction symmetrically.

## 3. Results

| Metric | Value | Interpretation |
|---|---|---|
| MAE | **0.041 m** (~4 cm) | On a series where full-record variation spans roughly ±0.2 m, this is a solid fit where the model tracks the trend closely |
| SMAPE | **46.3%** | Inflated by the fact that detrended monthly MSL values oscillate close to zero (small denominators blow up percentage error). The absolute error (MAE) is the more meaningful metric here. |

**Qualitative observations from the forecast plot:**
- The model correctly identifies and extrapolates the **long-term upward trend** in sea level at this station.
- It smooths over and misses short-term anomalies (storm surge events, seasonal current shifts), which is expected. Prophet is a trend/seasonality decomposition, not a physical ocean model.
- The uncertainty band widens appropriately the further the forecast extends past the training window, which is the behavior you want from a well-calibrated model.

**Comparison to NOAA regional scenarios:** NOAA's "Regional Scenarios" tool for this station provides multiple climate-model-informed projections (low to high emissions pathways) through 2100. The Prophet model's linear-ish extrapolation of the historical rate tends to track closest to NOAA's intermediate-low to intermediate scenarios, and since it has no way to account for potential acceleration in sea level rise from ice sheet dynamics, it can only extend the rate of change already present in the historical record.

## 4. Interpretation & Limitations

- **What this model is good for:** a fast, reproducible, statistically grounded first-pass forecast, and a way to sanity-check whether a station's historical trend is consistent with published regional projections.
- **What it isn't:** a substitute for physically-based climate models. Prophet cannot represent ice sheet melt dynamics, thermal expansion, or land subsidence, rather, it can only project forward whatever pattern already existed in the training window.
- **Training window matters:** using more historical data raises confidence in the long-term trend estimate, but can under-weight recent acceleration. Shorter, more recent windows are more responsive to recent trends but noisier and more sensitive to short-term anomalies. This tradeoff is the core experiment the notebook is set up to let you run (see the rolling-window and varying-training-length experiments).

## 5. Applying research skills from BNL to this project

Working on this project drew directly on research practices developed during my time at Brookhaven National Laboratory undergoing an AI/ML Data Science paid program, including:

- **Rigorous train/test methodology** : treating historical hold-out validation as non-negotiable before trusting a model's forward-looking output, the same way you wouldn't trust a physical model's predictions without validating it against known data.
- **Quantifying uncertainty, not just producing a point estimate** : reporting error metrics (MAE, SMAPE) and uncertainty intervals alongside the forecast itself, so the result is honest about its own limitations rather than presented as a single "answer."
- **Reproducibility** : building the pipeline as a documented, re-runnable notebook (rather than one-off analysis) so someone else, such as a classmate, a community member, a local planning board, can plug in their own station's data and get a trustworthy result without needing to understand the underlying statistics.
- **Pandas, Numpy, and other Python Coding skills** : Utilized the fundamental coding libraries of Python and the analytical regression skills taught at BNL, and how to analyze data visually and comparatively through code.

The intent behind open-sourcing this project is to make that same rigor available to non-specialists such as, local communities, students, or civic groups, who want a data-backed starting point for thinking about sea level rise in their area, without needing access to a national lab's modeling infrastructure. Anyone can fork this repo, swap in their own NOAA station, and get a station-specific forecast with honest error bars in under an hour.

## 6. Next steps

- Run the rolling-window and varying-training-length experiments described in the notebook to quantify the accuracy/recency tradeoff directly, rather than just discussing it qualitatively.
- Test the pipeline against 2–3 additional NOAA stations with different regional characteristics (e.g., a Gulf Coast station experiencing land subsidence vs. a West Coast station) to see how flexible the default Prophet configuration is.
- Consider adding Prophet's changepoint detection tuning to see if it better captures any acceleration in recent decades that a purely linear extrapolation would miss.
