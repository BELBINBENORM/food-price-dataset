# 🌾 Global Food Price Inflation & Hunger Index

Auto-updating pipeline that fetches, merges, and publishes a monthly food security panel to Kaggle every Monday — no manual work required after setup.

[![Update Food Price Dataset](https://github.com/BELBINBENORM/food-price-dataset/actions/workflows/update.yml/badge.svg)](https://github.com/BELBINBENORM/food-price-dataset/actions/workflows/update.yml)

---

## What This Repo Does

A GitHub Actions workflow runs every Monday at 07:00 UTC and:

1. Executes `notebook.ipynb` — fetches live data from FAO and World Bank
2. Writes six cleaned CSVs into `outputs/`
3. Pushes a new version to the Kaggle dataset automatically

No Kaggle session, no manual downloads, no maintenance.

---

## Output Files (published to Kaggle)

| File | Description | Earliest data |
|------|-------------|---------------|
| `fao_food_prices.csv` | FAO Food Price Index overall + 5 sub-indices (monthly) | Jan 1990 |
| `commodity_prices.csv` | Wheat, rice, maize, soybean oil + DAP, urea, potash spot prices | Jan 1960 |
| `crop_yields.csv` | Global wheat/rice/maize area, production, yield with YoY rates | 1961 |
| `food_security_indicators.csv` | Undernourishment %, food production index, cereal yield index | 1992 |
| `food_security_panel.csv` | Merged monthly panel + 6 derived food security ratios | Jan 1990 |
| `ml_features.csv` | ML-ready matrix: L1–L6 lags, rolling windows, crisis flags, targets | Jan 1990 |

---

## Data Sources

| Source | Data | Access |
|--------|------|--------|
| FAO FAOSTAT | Food Price Index + 5 sub-indices | Bulk ZIP / REST API |
| World Bank Pink Sheet | Monthly commodity + fertiliser spot prices | Excel download |
| FAO FAOSTAT (QCL) | Crop production + yield | Bulk ZIP |
| World Bank API | Annual food security indicators | JSON API, no key required |

All sources are free and open-access. No API keys required.

---

## Derived Metrics (food_security_panel.csv)

| Column | Formula | Interpretation |
|--------|---------|----------------|
| `cereals_affordability_idx` | `(100 / ffpi_cereals) × 100` | Below 100 = more expensive than 2014–2016 baseline |
| `dap_to_wheat_ratio` | `dap_usd_mt / wheat_usd_mt` | Input cost multiplier; >3.5 = farmer margin stress |
| `ffpi_volatility_12m` | 12m rolling σ of FFPI MoM change | High = shock regime; >3 = historically elevated |
| `supply_stress_flag` | Cereals YoY >20% AND fertiliser YoY >20% | 1 = compound supply shock (2008, 2011, 2022) |
| `food_shock_score` | Sum of binary stress flags (0–3) | 3 = maximum food crisis signal |
| `ffpi_crisis_flag` | `ffpi_overall > 130` | 1 = above 2011/2022 crisis threshold |

---

## ML Targets (ml_features.csv)

| Column | Type | Description |
|--------|------|-------------|
| `target_ffpi_chg_1m` | float | Next-month FFPI percent change (regression) |
| `target_ffpi_chg_12m` | float | Next-12-month FFPI percent change (regression) |
| `target_ffpi_up_12m` | int | 1 if FFPI higher in 12 months, else 0 (classification) |
| `target_cereals_chg_1m` | float | Next-month cereals index change (regression) |
| `target_shock_next_month` | int | 1 if food shock score ≥ 2 next month (classification) |

---

## Setup

### 1. Fork or clone this repo

```bash
git clone https://github.com/BELBINBENORM/food-price-dataset.git
```

### 2. Add Kaggle secrets

Go to **Settings → Secrets and variables → Actions** and add:

- `KAGGLE_USERNAME` — your Kaggle username
- `KAGGLE_KEY` — your Kaggle API key (from kaggle.com/settings)

### 3. Add dataset-metadata.json

```json
{
  "title": "Global Food Price Inflation & Hunger Index",
  "id": "your-kaggle-username/your-dataset-slug",
  "licenses": [{ "name": "CC0-1.0" }]
}
```

### 4. That's it

The workflow runs automatically every Monday. Trigger it manually anytime from the **Actions** tab.

---

## Resilience Notes

- **FAOSTAT Cloudflare blocking**: If FAO bulk ZIPs are unavailable (common in CI environments), the notebook automatically constructs a synthetic FFPI proxy by normalising World Bank wheat/maize/soybean-oil prices to the 2014–2016 = 100 base. The proxy faithfully captures the same price dynamics.
- **Fallback chain**: FAOSTAT ZIP → FAOSTAT REST API → synthetic proxy from WB Pink Sheet
- The pipeline will always produce output as long as the World Bank Pink Sheet is reachable.

---

## License

Code: MIT  
Data: CC0 — source data from FAO (CC BY-NC-SA 3.0 IGO) and World Bank (CC BY 4.0)
