# 04 — Data and Machine Learning Strategy

## 1. Purpose

This document defines how MandiMitra should collect, validate, store and use market and weather information, and how it should evaluate any price-estimation model before showing it to farmers.

The central rule is simple: **observed prices are facts reported by a source; forecasts are model estimates. They must never be blended or displayed as though they are the same thing.**

## 2. Candidate data sources

| Data | Candidate source | Potential use | Verification required |
|---|---|---|---|
| Daily mandi prices | Government of India OGD catalog entry generated through AGMARKNET | Historical and recent min/max/modal market prices | API/resource identifier, access method, coverage, update cadence, schema, license/terms and rate limits |
| Variety-wise mandi prices | OGD India resource for variety-wise daily market prices | Distinguish commodity varieties where available | Field meanings, completeness, stable identifiers and update behavior |
| Weather observations/history | Open-Meteo Historical Weather API or another approved provider | Optional explanatory features and contextual market analysis | Terms for intended usage, location granularity, historical coverage, attribution and operational limits |
| Weather forecast | Approved weather forecast API | Optional short-horizon context | Forecast horizon, update cadence, accuracy, terms and reliability |
| Market/commodity reference data | Verified government or operational reference source | Consistent state, district, mandi, commodity and variety mapping | Canonical identifiers and update governance |
| Buyer/farmer listing data | User submitted | Supply, quantity, availability, bids and listing state | Validation, moderation, consent and audit |

**Source notes:** The OGD India catalog describes daily wholesale minimum, maximum and modal prices generated through AGMARKNET. Its related variety-wise resource lists fields including state, district, market, commodity and variety. The catalog/resource pages must be rechecked at implementation time because access methods and metadata can change.

- [OGD India — Current daily mandi prices](https://data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi)
- [OGD India — Variety-wise daily market prices](https://data.gov.in/resource/variety-wise-daily-market-prices-data-commodity)
- [Open-Meteo historical weather API documentation](https://open-meteo.com/en/docs/historical-weather-api)

No integration should be described as operational until a developer has successfully retrieved the relevant records, recorded the request/response contract, checked usage terms and established a repeatable ingestion job.

## 3. Data acquisition workflow

~~~mermaid
flowchart TD
  A[Scheduled ingestion] --> B[Provider adapter]
  B --> C[Raw response capture]
  C --> D[Schema validation]
  D --> E{Valid record?}
  E -- No --> F[Quarantine / rejection log]
  E -- Yes --> G[Normalize identifiers and units]
  G --> H[Deduplicate]
  H --> I[Persist immutable observation]
  I --> J[Quality checks and coverage metrics]
  J --> K[Feature dataset]
  K --> L[Model training and backtesting]
~~~

### Ingestion run record

Every run should persist:

- Provider and resource identifier.
- Run ID and software/adapter version.
- Request time and retrieval time.
- Source date range and market filters.
- Total records received, accepted, rejected and deduplicated.
- Rejection reasons and schema warnings.
- Data checksum or equivalent repeatability marker.
- Run status: started, succeeded, partial or failed.

### Idempotency

Use a stable natural key where source fields permit it, such as provider + source market identifier + commodity + variety + grade + observation date + unit. If the upstream source has a stable row identifier, preserve it. Repeated downloads must not create duplicate observations.

## 4. Canonical market observation model

Each normalized observation should retain:

| Field | Meaning |
|---|---|
| observation_date | Date to which the market price applies |
| state / district | Canonical location hierarchy |
| market_id | Canonical mandi reference |
| commodity_id | Canonical commodity reference |
| variety_id | Nullable variety reference |
| grade | Source grade or explicit unknown value |
| min_price | Reported minimum price |
| max_price | Reported maximum price |
| modal_price | Reported modal price |
| currency | Currency code, initially INR |
| price_unit | Unit used by source, such as quintal or kilogram |
| arrivals_quantity | Nullable arrivals quantity if source supplies it |
| arrivals_unit | Unit for arrivals |
| source_name | Provider identity |
| source_record_id | Provider record identifier where available |
| retrieved_at | When MandiMitra fetched the source row |
| quality_status | Accepted, warning, rejected or corrected-with-audit |

Never assume that two price records are comparable until commodity, variety/grade where applicable, currency and unit have been reconciled.

## 5. Quality checks

### Structural validation

- Required market, commodity, date and price fields are present.
- Date parses and is within permitted bounds.
- Numeric values are finite and non-negative unless source semantics explicitly allow another interpretation.
- Minimum price does not exceed maximum price.
- Modal price is checked against the min/max range; violations are flagged, not silently rewritten.
- Currency and unit values map to supported canonical values.
- Market and commodity names map to known references or enter a review queue.

### Temporal and duplicate validation

- Repeated ingestion of the same source row is idempotent.
- Late-arriving source rows are accepted with original observation date and new retrieval timestamp.
- Corrections are traceable and preserve prior versions or audit history.
- Future-dated observations are quarantined unless explicitly explained by source semantics.
- Missing dates are represented as missing, not zero or fabricated values.

### Statistical checks

- Flag large changes against recent history for review.
- Check price distributions by commodity, market, unit and date.
- Treat statistical outliers as warnings, not automatic deletion rules.
- Maintain rejected-row samples for debugging without leaking personal data.

## 6. Forecasting scope

### MVP prediction target

Predict the **modal mandi price** for a selected commodity-market combination over a short, explicitly stated horizon. This target is provisional and must be checked against actual source coverage and data consistency.

Suggested initial horizons for experimentation:

- Next available market day.
- Three calendar days ahead, only where the target definition is meaningful.
- Seven days ahead, only if backtesting supports this horizon.

The app should show only horizons that meet the configured data sufficiency and quality thresholds. It must not fabricate a market-day price for a day when no observation is expected.

### Candidate features

| Feature family | Examples | Guardrail |
|---|---|---|
| Price history | Previous 1/3/7/14/28 observed prices | Build only from information available at prediction time |
| Rolling statistics | 7/14/28-day median and mean, volatility | Exclude future rows and account for missing days |
| Calendar | Day of week, month, season, festival/event indicator when verified | Do not assume a festival effect without evidence |
| Market and crop | Market ID, commodity, variety, grade, unit | Encode categories and handle unseen categories |
| Supply | Arrival quantity and recent change where reliable | Missing arrivals remain missing; do not assume zero |
| Weather | Recent and forecast precipitation/temperature where relevant | Use only valid issue-time weather data; do not imply causation |
| Cross-market context | Neighboring market prices and spread | Normalize product, date and unit before comparison |

Features such as historical arrivals or weather must be used only when they are available at the exact forecast issue time. Data leakage from future observations invalidates model evaluation.

## 7. Modeling approach

Use a measured progression rather than starting with a complex model.

| Stage | Candidate | Purpose |
|---|---|---|
| Baseline 1 | Last observed price | Minimum benchmark |
| Baseline 2 | Same weekday / seasonal naive where enough history exists | Capture recurring patterns |
| Baseline 3 | Rolling median | Robust simple baseline |
| Model 1 | Regularized linear model or tree-based model | Establish non-linear signal |
| Model 2 | LightGBM / XGBoost | Candidate tabular forecasting model if data size and license permit |
| Later research | Global multi-market model or time-series architecture | Only after sufficient clean coverage |

Do not assume a neural network is superior. Choose the simplest method that improves out-of-sample performance and remains understandable and operationally maintainable.

## 8. Evaluation protocol

### Time-aware validation

- Split by time, never randomly across rows.
- Use rolling-origin evaluation: train on the past, validate on the future, then advance the cutoff.
- Keep a final holdout period untouched until model selection is complete.
- Evaluate by commodity, market, forecast horizon and data-density segment.
- Compare every candidate with the same naive baselines.
- Prevent features from using information published after forecast issue time.

### Metrics

| Metric | Use |
|---|---|
| MAE | Average absolute error in the market price unit |
| WAPE | Scale-aware aggregate error; define behavior when total actual is zero |
| sMAPE | Relative comparison with more symmetry than MAPE, with documented zero handling |
| Pinball loss | Evaluate forecast quantiles |
| Prediction interval coverage | Check whether actual values fall inside the stated interval at the expected rate |
| Baseline skill | Quantify whether the model improves on the chosen naive benchmark |

Report median and distribution of error, sample count and missing-data coverage. A single overall accuracy percentage is not sufficient.

### Release criteria

A forecasting model can be enabled only when:

1. It beats an agreed baseline on the predeclared evaluation segments.
2. Performance is stable enough for the target commodity/market/horizon.
3. Interval calibration is measured and documented.
4. Data freshness and minimum-history rules are enforced.
5. The model version, training window, feature code and metrics are reproducible.
6. Product and domain reviewers approve the wording shown to farmers.

If the model does not meet these conditions, continue to show observed prices and historical trends without the forecast.

## 9. Forecast response contract

A forecast record should include:

- market_id, commodity_id, optional variety_id and price_unit.
- target_date and horizon.
- point estimate or median estimate.
- lower and upper interval values with their quantile meaning.
- model_name and model_version.
- generated_at and data_through_date.
- training_window_start and training_window_end.
- quality_status and a human-readable limitation reason if unavailable.

The client should visually label it as an estimate and show the date and range. Do not display a false precision that the training data cannot support.

## 10. Example farmer-facing language

**Appropriate:** “Estimated modal price for the selected market day: ₹X–₹Y per quintal. Based on data through [date]. Estimates can differ from actual trading prices.”

**Not appropriate:** “You will receive ₹Y” or “This crop will definitely increase in price.”

## 11. Monitoring and retraining

Monitor:

- ingestion freshness and row counts;
- market/commodity coverage and missingness;
- feature distribution shifts;
- prediction distribution and model error as delayed actuals arrive;
- performance by commodity, mandi and horizon;
- model artifact and inference failures.

Retraining should be triggered by documented evidence and validation, not by a fixed schedule alone. Retain model artifacts, evaluation reports and a rollback version. A model can be disabled independently of the market-price feature.

## 12. Data governance

- Record source attribution and applicable usage terms.
- Retain raw source data only as permitted and necessary.
- Keep normalized records linked to source provenance.
- Restrict access to raw datasets and model artifacts.
- Document manual corrections and who approved them.
- Do not mix user-submitted listing prices with official observed mandi prices.
