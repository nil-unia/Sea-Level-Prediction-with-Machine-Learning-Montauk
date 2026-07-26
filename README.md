# 🌊 Sea Level Prediction with Machine Learning

Forecasting long-term sea level trends using NOAA tide gauge data and Facebook's **Prophet** time series model — built as a Google Colab notebook that anyone can adapt to their own coastal station.

## Overview

Coastal communities need reliable, forward-looking estimates of sea level rise to plan infrastructure, flood mitigation, and climate adaptation. This project uses historical monthly mean sea level (MSL) records from [NOAA's Sea Level Trends](https://tidesandcurrents.noaa.gov/sltrends/sltrends.html) database to train a Prophet forecasting model, evaluate it against held-out historical data, and project sea levels out to the year 2100.

This repo currently uses **Montauk, NY** as the reference station, with monthly MSL data spanning **1947–2026**, but the notebook is written so any NOAA station with a long enough record (100+ years recommended) can be swapped in.

## Why Prophet?

Prophet (developed by Meta) is well-suited to this problem because sea level data is:
- **Trending** — sea levels are rising over the long term, not stationary
- **Seasonal** — there's a repeating annual cycle layered on top of the trend
- **Noisy** — short-term fluctuations (storms, currents, measurement variance) obscure the signal

Prophet decomposes the series into trend + seasonality + noise automatically and produces uncertainty intervals alongside its point forecasts, which makes it easy to communicate *how confident* a projection is — not just what it predicts.

## What's in this repo

| File | Description |
|---|---|
| `sea_level_prediction.ipynb` | The full Colab notebook: data loading, preprocessing, visualization, train/test split, Prophet model training, evaluation, and long-range (2100) forecasting |
| `sea_level_prediction.csv` *(user-supplied)* | Cleaned NOAA station export — not included in this repo; see setup below |
| `ANALYSIS.md` | Write-up of the modeling approach, results, and interpretation |

## Pipeline

1. **Data collection** — Export monthly MSL data for a chosen NOAA station, strip the header rows, and clean column names.
2. **Preprocessing** — Combine `Month`/`Year` into a proper `ds` datetime column (Prophet's required format) and set it as the index.
3. **Exploratory visualization** — Scatter plots and 12-month rolling averages to inspect long-term trend, seasonality, and outliers before modeling.
4. **Train/test split** — Date-based split (default: train 1975–2000, test 2000–2026) rather than a random split, since this is a time series problem.
5. **Model training** — Fit a Prophet model on the training window.
6. **Evaluation** — Forecast the test window and score it with **MAE** and **SMAPE**, then visually compare forecast vs. actual with uncertainty bands.
7. **Long-range projection** — Extend the trained model out to 2100 and compare it against NOAA's own published regional sea level rise scenarios.

## Getting started

1. **Google account** — required for Colab and Drive.
2. **Get the data** — Go to the [NOAA Sea Level Trends page](https://tidesandcurrents.noaa.gov/sltrends/sltrends.html), pick a station (100+ years of record recommended), and export its data to CSV.
3. **Clean the CSV** — Remove the first five metadata rows, verify there's no whitespace in column headers, save as `sea_level_prediction.csv`.
4. **Set up Drive** — Create a folder named `sea_level_prediction` in your Google Drive, and upload both the notebook and the CSV into it.
5. **Run it** — Open the notebook in Colab, run the *Importing Libraries* cell first, then work through the notebook top to bottom.

> Full step-by-step instructions (with screenshots) are in the companion Science Buddies procedure this project is based on.

## Key results (Montauk station)

- **MAE:** ~0.041 m — on average, forecasts differ from actual monthly sea level by about 4 cm
- **SMAPE:** ~46.3% — a reminder that percentage error metrics can look large for series that oscillate near zero even when the absolute error is small
- The model correctly captures the long-term upward trend, but (as expected) smooths over short-term storm- and current-driven anomalies

See `ANALYSIS.md` for full interpretation, limitations, and comparison against NOAA's own regional projections.

## Experimenting further

The notebook is set up to make it easy to test:
- **Different training windows** (e.g., rolling 25-year windows, or short vs. long history) to see how training length affects forecast accuracy
- **Different stations**, to compare regional rates of sea level rise
- **Prophet hyperparameters** (seasonality mode, changepoint sensitivity) for more advanced experimentation

## Limitations

- Prophet is a statistical trend/seasonality model — it doesn't know about physical processes like ice sheet dynamics, thermal expansion, or land subsidence, so long-range (2100) forecasts should be treated as an extrapolation, not a physical simulation.
- Longer historical records tend to produce more stable long-term trend estimates, but may under-weight recent acceleration in sea level rise.
- Regional NOAA scenarios (which incorporate physical climate models) are a better source for planning-grade projections; this model is best used as a data science exercise and a first-pass comparison point.

## Acknowledgments

Project structure and procedure adapted from [Science Buddies: "Can Machine Learning Forecast Future Sea Levels?"](https://www.sciencebuddies.org/science-fair-projects/project-ideas/ArtificialIntelligence_p031/artificial-intelligence/sea_level). Data sourced from NOAA Tides & Currents.
