# Indonesia Hospital Bed Need Forecast

Predicting province-level hospital bed shortages in Indonesia by combining
historical population growth, socioeconomic indicators, and current hospital
capacity data — with an interactive tool to project future bed needs by year.

## Project Goal

For each of Indonesia's provinces, this project answers three questions:

1. **How many hospital beds does each province currently have, relative to its population?**
2. **Which provinces are in critical need of additional beds today?**
3. **Given a target year, how many beds will that province need — and is current capacity enough?**

The end deliverable is an interactive tool: input a year, get a projected
population, projected bed requirement, current capacity, and a gap/surplus
flag for every province.

## Data Sources

| Source | Description | Granularity |
|---|---|---|
| [Hospital Data in Indonesia (Kaggle)](https://www.kaggle.com/datasets/muhammadhabibna/hospital-data-in-indonesia) | Per-hospital records: beds, workforce, services, class, ownership | Hospital-level |
| [Population by Province (BPS, multi-year)](https://www.bps.go.id/) | Yearly population counts and growth rates by province | Province-year |
| [IPM — Human Development Index by Province](https://data.go.id/dataset/dataset/indeks-pembangunan-manusia-menurut-provinsi-2010-2017) | Composite health/education/living-standard index | Province-year |
| [Rata-rata Pengeluaran per Kapita (BPS)](https://www.bps.go.id/) | Average monthly per-capita expenditure, urban areas | Province-year |
| [Migrasi Risen (BPS)](https://www.bps.go.id/) | Recent-migration statistics by province | Province-census year |
| Population 2022 (BPS) | Province population baseline, 2022 | Province |

**Note on population data:** BPS population figures are a mix of actual
census counts (years ending in 0) and official projections for intervening
years, drawing on different projection vintages (2010-2035, 2015-2045,
2020-2050) depending on the year. This is documented per-year in the
cleaning notebooks where relevant, since it affects how forecasted years
should be interpreted.

## Methodology

The forecasting problem is split into two stages rather than a single model,
since a single cross-sectional regression can't meaningfully capture
population growth over time with limited yearly data points per province:

**Stage 1 — Population Forecasting (per province)**
A trend/growth-rate model fit per province on historical population data,
validated by holding out recent years and comparing against actuals.

**Stage 2 — Explanatory Regression**
Growth rate ~ IPM + migration + expenditure, across provinces — this
explains *why* growth rates differ, rather than being used as a forecasting
input. Given the small sample size (34-38 provinces), results are reported
with confidence intervals and explicit caveats about statistical power,
rather than overclaiming significance.

**Stage 3 — Bed-Need Calculation**
```
beds_needed(province, year) = projected_population(province, year) × benchmark_ratio − current_beds(province)
```
A positive result flags that province as facing a projected deficit.

## Project Structure

```
hospital-bed-forecast/
├── data/
│   ├── raw/                          # untouched downloads (gitignored)
│   └── processed/                    # cleaned, province-level CSVs
├── notebooks/
│   ├── 01a_cleaning_hospital.ipynb
│   ├── 01b_cleaning_population.ipynb
│   ├── 01c_cleaning_ipm.ipynb
│   ├── 01d_cleaning_expenditure.ipynb
│   ├── 01e_cleaning_migration.ipynb
│   ├── 02_merge_and_eda.ipynb
│   └── 03_modeling.ipynb
├── src/                               # reusable cleaning/modeling functions
├── app/                                # Streamlit interactive calculator
├── requirements.txt
└── README.md
```

## Setup

```bash
# clone the repo
git clone https://github.com/YOUR_USERNAME/hospital-bed-forecast.git
cd hospital-bed-forecast

# create and activate a virtual environment
python -m venv venv
venv\Scripts\activate.bat        # Windows (cmd)
# or: venv\Scripts\Activate.ps1  # Windows (PowerShell)
# or: source venv/bin/activate   # macOS/Linux

# install dependencies
pip install -r requirements.txt
```

### Getting the data

Raw data files are **not** committed to this repo (see `.gitignore`) due to
size and licensing. Download each source from the links above and place
them in `data/raw/` before running the cleaning notebooks. See each
notebook's first cell for expected filenames.

## Data Cleaning Notes

Key decisions made during cleaning are documented inline as markdown in
each notebook. Notable ones:

- **Hospital bed counts >2,000 were treated as data errors**, not real
  outliers — verified against Indonesia's largest hospital (RSCM,
  ~1,542 beds) as a real-world ceiling.
- **Rows with implausibly high beds but near-zero workforce** (e.g., 1,220
  beds / 20 staff) were identified via a beds-vs-workforce scatter plot and
  dropped as likely scraping/entry errors.
- **The `kelas` (hospital class) column was found to be internally
  inconsistent** with Indonesia's official classification standards (e.g.,
  Class B facilities showing higher bed counts than Class A minimums) and
  was therefore **not used** to validate or adjust bed-count values.

## Current Status

- [x] Project setup (venv, git, structure)
- [x] All 6 raw data sources acquired
- [x] Hospital dataset cleaned, outliers resolved, aggregated to province level
- [ ] Population panel cleaned (multi-year, in progress — filling gap years)
- [ ] IPM, expenditure, migration datasets cleaned
- [ ] Merged province-level EDA
- [ ] Stage 1 & 2 modeling
- [ ] Bed-need calculator
- [ ] Streamlit app
- [ ] Final writeup

## Limitations

- Cross-sectional regression across 34-38 provinces has limited statistical
  power; results should be read as exploratory/directional, not causal.
- Population figures for non-census years are official BPS projections,
  not measured counts — this affects the independence of any trend-fitting
  done on top of them.
- Hospital data quality issues (see cleaning notes) mean some rows were
  excluded rather than imputed; this is a Kaggle-community-sourced dataset,
  not an official government release, and may not perfectly match official
  Kemenkes figures.

## Tech Stack

Python · pandas · numpy · scikit-learn · matplotlib · seaborn · Streamlit ·
Jupyter
