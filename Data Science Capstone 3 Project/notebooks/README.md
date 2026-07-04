
## Accelerating Electric Vehicle Adoption: A County-Level Predictive Framework for Identifying High-Impact Regions

**Springboard Data Science Career Track — Capstone 3**
<a target="_blank" href="https://cookiecutter-data-science.drivendata.org/">
    <img src="https://img.shields.io/badge/CCDS-Project%20template-328F97?logo=cookiecutter" />
</a>

## Overview

Electric vehicle (EV) adoption is growing rapidly across the United States, but growth is highly uneven — even between counties in the same state. This project builds a data-driven framework to quantify EV adoption at the county level in Washington State, identify the socioeconomic and infrastructure factors that drive it, and segment counties into actionable investment/policy tiers.

The project combines two modeling approaches on 2017–2025 Washington State county-year data:

- **Ridge Regression** — quantifies which features drive EV adoption rate and by how much
- **K-Means Clustering** — segments counties into groups with distinct adoption profiles and constraints

Together, these outputs support a **three-phase infrastructure and policy prioritization matrix** for Washington's 39 counties.

## Business Problem

Conventional (fossil-fuel) transportation contributed roughly 28% of U.S. greenhouse gas emissions in 2022, making EV adoption a key lever for climate goals. But charging infrastructure and incentive programs are expensive, and misallocating them to low-impact regions wastes public and private capital. This project uses Washington State as a case study for a scalable, reusable framework that identifies **where** to invest and **why** adoption lags in certain regions.

**Success criteria:**
- A regression model with meaningful explanatory power that identifies significant drivers of county-level EV adoption
- Interpretable, stable clustering that segments counties into policy-relevant groups
- Actionable infrastructure/policy recommendations grounded in both models

## Data Sources

All data is Washington State public data, aggregated to county/year granularity (2017–2025):

| Source | Description |
|---|---|
| [Electric Vehicle Population Data](https://data.wa.gov/Transportation/Electric-Vehicle-Population-Data/f6w7-q2d2/about_data) | Washington DOL registered EV records (make, model, type, range, CAFV eligibility) |
| [Vehicle Registrations by Class and County](https://data.wa.gov/Transportation/Vehicle-Registrations-by-Class-and-County/hmzg-s6q4/about_data) | All passenger vehicle registrations, used as the adoption-rate denominator |
| [Median Household Income Estimates](https://ofm.wa.gov/data-research/economy/median-household-income-estimates/) | County-level median income |
| [Population Estimates](https://ofm.wa.gov/data-research/population-demographics/estimates/april-1-official/) | County-level population |
| [EV Charging Stations Data](https://ev-map-wsdot.hub.arcgis.com/apps/7e310dcd476640ec8c611c101f610c09/explore) | Public EV charging station locations/counts |

The median income, population, and charging station datasets were pre-merged into a single supporting CSV prior to this project's data wrangling step.

## Repository / Notebook Structure

| Notebook | Purpose |
|---|---|
| `Data_Science_Capstone_3_-_Project_Proposal.docx` | Original project proposal: problem statement, scope, stakeholders, approach |
| `02_data_wrangling.ipynb` | Loads, cleans, and merges all raw datasets into a single county-year analytical dataset; engineers the target variable and initial features |
| `03_Exploratory_Data_Analysis.ipynb` | Univariate/bivariate/time-trend/correlation analysis; feature transformation and selection; produces the model-ready dataset |
| `04_Preprocessing_and_Training.ipynb` | Time-aware train/test split, feature scaling, and dataset preparation for both modeling paths |
| `05_Modeling.ipynb` | Ridge regression (adoption drivers) and K-Means clustering (county segmentation), plus combined insights and recommendations |

## Methodology

### 1. Data Wrangling
Three raw sources (EV registrations, all vehicle registrations, socioeconomic/infrastructure data) were cleaned, standardized, and merged to the county-year grain (2017–2025, 39 WA counties). Key steps:
- Filtered EV data to Washington State only; dropped unneeded geographic/political fields
- Imputed a small number of missing `Electric Range` values (median by make/model, with one external lookup for a model with no comparable records)
- Aggregated EV records to county-year level (BEV/PHEV mix, CAFV eligibility rate, EV count)
- Aggregated all-vehicle registrations to county-year totals and EV-only subtotals
- Merged all three sources and engineered the target variable and supporting features:
  - **`ev_adoption_rate`** (target): registered EVs ÷ total registered passenger vehicles, ×100
  - `ev_growth_rate`, `ev_adoption_lag` (strictly prior-year, to avoid temporal leakage), `income_per_vehicle`, `ev_charging_density`, and related flags
- Dropped `bev_avg_range`/`phev_avg_range`/`range_missing_rate` after discovering WA stopped tracking BEV electric range after 2020 (a data availability issue, not a modeling choice)

### 2. Exploratory Data Analysis
- **Distributions:** `ev_adoption_rate`, population, and charging density/stations were all right-skewed and required transformation
<img src="reports/figures/all_ev_adoption_related_distributions.png" alt="EV Adoption Related Distributions" width="400" />
<img src="reports/figures/all_socioeconomic_distributions.png" alt="Socioeconomic Distributions" width="400" />
<img src="reports/figures/all_charging_distributions.png" alt="EV Charging Distributions" width="400" />
<img src="reports/figures/all_ev_feature_distributions.png" alt="EV Feature Distributions" width="400" />
- **Bivariate analysis:** strongest positive relationships with adoption rate were prior-year adoption (lag), median income, and charging infrastructure (density and station count); vehicle-technology features (CAFV eligibility, BEV share) were weak predictors

- **Correlation with target (`ev_adoption_rate`):**

  | Feature | Correlation |
  |---|---|
  | ev_adoption_lag | 0.99 |
  | median_income | 0.77 |
  | ev_charging_density | 0.63 |
  | ev_charging_stations | 0.58 |
  | population | 0.43 |
  | cafv_eligibility_rate | 0.31 |
  | bev_ratio_within_ev | 0.24 |
  | ev_growth_rate | 0.07 |
  | income_per_vehicle | -0.17 (distorted by rural outliers) |
  | new_ev_purchase_ratio | -0.36 (distorted by reporting lag) |

  EV Adoption Lag, Median Income  & EV Charging Density features seem to have the highest correlation with EV Adoption.

 <img src="reports/figures/ev_raw_correlations.png" alt="EV Raw Correlations Matrix" width="400" />

 The boxplots reveal a clear socio-economic divide where high-adoption regions are clearly separated by higher median incomes, charging station densities, and historical adoption momentum, while highlighting extreme population outliers and the heavy visual redundancy of the vehicle-mix features such as cafv_eligibility_rate & bev_ratio_within_ev (showing compressed, high-positioned medians for high-adoption zones).

 <img src="reports/figures/ev_segmentation_box_plots.png" alt="EV Segmentation Box Plots" width="400" />

- **Time trends:** adoption, adoption lag, and charging infrastructure all show accelerating (near-exponential) growth from 2017–2025; income and population grow steadily

<img src="reports/figures/all_ev_feature_trends.png" alt="EV Features Trends" width="400" />

The top 5 vs. bottom 5 counties EV Adoption trends (below) show a huge divide where high ev adoption counties have achieved an exponential momentum while low adoption counties are seeing 0 to extremely low adoption potentially due to low population, lack of income, infrastructure and policies.

<img src="reports/figures/all_ev_top_adoption.png" alt="EV Adoption - Top Counties" width="400" />

<img src="reports/figures/all_ev_bottom_adoption.png" alt="EV Adoption - Bottom counties" width="400" />


- **Transformation:** Yeo–Johnson power transform applied to population, median income, charging density, adoption lag, and the target — chosen over log transform because it better handled the zero-heavy, right-skewed distributions. VIF checks confirmed low multicollinearity (all values < 4.3) across the retained feature set.
- Weakly correlated / anomaly-prone features (`income_per_vehicle`, `new_ev_purchase_ratio`, `bev_ratio_within_ev`, `cafv_eligibility_rate`, `ev_charging_stations`, `ev_growth_rate`) were dropped from the modeling dataset.

<img src="reports/figures/yj_transformed_correlations.png" alt="EV Transformed Key Feature Correlations Matrix" width="400" />

### 3. Preprocessing & Training Setup
- Two feature sets were finalized:
  - **Linear Regression features:** `yj_population`, `yj_median_income`, `yj_ev_adoption_lag`, `yj_ev_charging_density`, `has_charging_infrastructure`, `lag_missing_flag`
  - **K-Means features:** `yj_population`, `yj_median_income`, `yj_ev_charging_density`, `has_charging_infrastructure`
- `county` was excluded as a model feature (to avoid memorizing specific counties / geography-driven clusters) but retained as a reference column
- **Time-aware split:** train = 2017–2023, test = 2024–2025 (~80/20), since the data is a panel/time series and a random split would leak future information
- Features were standardized with `StandardScaler` (fit on train only for the regression set)
- No train/test split was used for K-Means (unsupervised); the target was retained only as a label for post-hoc interpretation

### 4. Modeling

**Ridge Regression** (chosen over OLS for its regularization, given a small ~78-row test set)
- Hyperparameter `alpha` tuned via `GridSearchCV` with `TimeSeriesSplit` (5 folds) to respect temporal ordering
- **Test set (2024–2025) performance:** R² ≈ 0.96, RMSE ≈ 0.14, MAE ≈ 0.11 (on the Yeo–Johnson-transformed adoption rate)
- **Feature importance (standardized coefficients):** prior-year adoption (lag) is by far the dominant driver, followed by median income (slightly ahead of charging infrastructure), with population contributing only weakly

<img src="reports/figures/ridge_feature_imp_plot_w_population.png" alt="EV Ridge Feature Importance Plot" width="400" />

- **Residuals:** roughly centered at zero and approximately normal, with a mild tendency to over-predict at higher adoption values — acceptable but not perfect adherence to linear regression assumptions

<img src="reports/figures/residuals_distribution.png" alt="Ridge Model Residuals Distribution" width="400" />

<img src="reports/figures/residuals_vs_predictions.png" alt="Ridge Model Residuals Vs. Predictions Plot" width="400" />

**K-Means Clustering** (county-level, latest available year per county)
- Optimal k = 3, selected via elbow method and silhouette score
- **Cluster 0 — High Adoption (16 counties, 41%):** ~3% avg. adoption, high income (~$96K), high population, relatively low public charging density (~0.03) — adoption driven by purchasing power and likely home charging
- **Cluster 1 — Medium Adoption (22 counties, 56%):** ~1.6% avg. adoption, mid income (~$60K–95K), lower population, higher public charging density (~0.04) — public infrastructure is substituting for lower income
- **Cluster 2 — Rural Isolation Outlier (1 county — Garfield):** near-zero adoption, moderate income (~$71K), zero public charging infrastructure, small population — a structural "charging desert"

<img src="reports/figures/kmeans_clusters.png" alt="KMeans Clusters" width="400" />


### 5. Combined Insights
- Income sets the initial adoption trajectory, but the strong lag/momentum effect means early movers compound their advantage over time
- Charging infrastructure can partially substitute for lower income (Cluster 1), acting as an equalizer
- Without *any* infrastructure, it is difficult to build EV Adoption momentum, regardless of income (Cluster 2 / Garfield)

## Recommendations: Three-Phase Prioritization Matrix

| Phase | Priority | Target Counties | Strategy | Funding |
|---|---|---|---|---|
| **1 — Foundation** | 🔴 Critical | Garfield | Seed baseline highway-corridor charging to eliminate WA's only total charging desert | 100% public capital (NEVI / state clean energy grants) |
| **2 — Activation** | 🟡 Demand activation | Cluster 1 counties (San Juan, Jefferson, Kittitas, Chelan, and other lower-income, infrastructure-rich counties) | Vehicle purchase rebates and used-BEV subsidies to overcome affordability barriers; maintain/enforce DC fast-charger uptime | Point-of-sale rebates, subsidies, public–private matching |
| **3 — Autonomy** | 🟢 Sustaining market | Cluster 0 counties (King, Snohomish, Clark, Pierce, and other high-income, high-population counties) | Rely on private commercial capital for destination charging; limit public spend to grid-capacity matching and select low-income pockets | Mostly private/commercial capital |

## Tech Stack

- **Language:** Python (pandas, numpy)
- **Visualization:** matplotlib, seaborn, plotly
- **Modeling:** scikit-learn (`Ridge`, `KMeans`, `GridSearchCV`, `TimeSeriesSplit`, `Pipeline`, `StandardScaler`, `PowerTransformer`)
- **Diagnostics:** `statsmodels` (variance inflation factor)

## Limitations & Assumptions

- Analysis is limited to Washington State; the framework is designed to be portable but has not been validated on other states
- EV adoption rate is a constructed label (registered EV ÷ total registered vehicles), not a directly reported metric
- Electric range tracking for BEVs was discontinued in state data after 2020, so range-based features could not be used
- New EV purchase ratio and income-per-vehicle showed data artifacts (registration reporting lags, rural outlier distortion) and were excluded rather than corrected
- The K-Means model uses each county's most recent year only, so it reflects a 2025 snapshot rather than trajectory over time
- Sample size is modest (39 counties × up to 9 years), and one cluster is a single-county outlier — results should be interpreted directionally rather than as precise point estimates

## Future Work

- **Geographic generalization:** validate the framework on additional states to test portability and separate state-specific drivers from broadly generalizable ones
- **Finer spatial granularity:** move from county-level to ZIP-code or census-tract level analysis where data allows
- **Charging infrastructure detail:** differentiate charger type (Level 2 vs. DC fast) and station reliability/uptime, rather than treating all infrastructure as equivalent
- **Time-series forecasting:** extend beyond a static drivers model to explicitly forecast adoption several years forward (e.g., ARIMA, panel-data models)
- **Model refinement:** test non-linear models (gradient boosting, random forest) to capture the mild residual non-linearity seen in the Ridge model, weighing interpretability trade-offs
- **Deployment:** package the framework as an interactive dashboard (eg. Tableau) for policymakers to explore, refreshed as new registration data becomes available

## Deliverables

- Cleaned, merged, model-ready datasets (data wrangling → EDA → preprocessing)
- Ridge regression model quantifying drivers of EV adoption
- K-Means segmentation of WA counties into 3 policy-relevant clusters
- Three-phase infrastructure/policy prioritization matrix
- Full documentation of methodology, findings, and business recommendations (this README and the accompanying project report)
## Project Organization

```
├── LICENSE            <- Open-source license if one is chosen
├── Makefile           <- Makefile with convenience commands like `make data` or `make train`
├── README.md          <- The top-level README for developers using this project.
├── data
│   ├── external       <- Data from third party sources.
│   ├── interim        <- Intermediate data that has been transformed.
│   ├── processed      <- The final, canonical data sets for modeling.
│   └── raw            <- The original, immutable data dump.
│
├── docs               <- A default mkdocs project; see www.mkdocs.org for details
│
├── models             <- Trained and serialized models, model predictions, or model summaries
│
├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
│                         the creator's initials, and a short `-` delimited description, e.g.
│                         `1.0-jqp-initial-data-exploration`.
│
├── pyproject.toml     <- Project configuration file with package metadata for 
│                         data_science_capstone_3_project and configuration for tools like black
│
├── references         <- Data dictionaries, manuals, and all other explanatory materials.
│
├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures        <- Generated graphics and figures to be used in reporting
│
├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
│                         generated with `pip freeze > requirements.txt`
│
├── setup.cfg          <- Configuration file for flake8
│
└── data_science_capstone_3_project   <- Source code for use in this project.
    │
    ├── __init__.py             <- Makes data_science_capstone_3_project a Python module
    │
    ├── config.py               <- Store useful variables and configuration
    │
    ├── dataset.py              <- Scripts to download or generate data
    │
    ├── features.py             <- Code to create features for modeling
    │
    ├── modeling                
    │   ├── __init__.py 
    │   ├── predict.py          <- Code to run model inference with trained models          
    │   └── train.py            <- Code to train models
    │
    └── plots.py                <- Code to create visualizations
```

--------

