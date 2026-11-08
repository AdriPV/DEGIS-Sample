# Evaluating the 2021 Chicago Speed Camera Policy

## Research Question
Did the March 2021 policy change reduce daily speed camera violations in Children's Safety Zones in Chicago?

## Data Source
Data was downloaded from the [Chicago Data Portal](https://data.cityofchicago.org/) on 2022.07.20. The dataset used is "Speed Camera Violations".

## Methodology
1.  **Data Cleaning:** Violations were aggregated to a monthly level. Only cameras that were active before 2018 were retained to establish a consistent baseline cohort.
2.  **Exploratory Analysis:** A time-series line graph was generated to visualize monthly violation trends from 2018 to 2021.
3.  **Causal Analysis:** A Two-Way Fixed Effects (FE) regression model was estimated to evaluate the policy's impact. The model includes camera-specific fixed effects (to control for time-invariant differences between cameras) and month-specific fixed effects (to control for seasonality).

## Key Findings
*   The coefficient for the 'post' policy variable was negative and statistically significant at the 5% level, suggesting the policy was associated with a reduction in monthly violations.

## Repository Structure
- `data/`: Contains the raw CSV file.
- `scripts/01_analysis.R`: The R script for data cleaning, visualization, and regression.
- `output/`: Contains the generated time-series plot (`.png`) and the regression summary (`.txt`).

## Limitations
- The model does not cluster standard errors, which may understate uncertainty.
- A formal parallel trends test (event study) was not performed; therefore, the results should be interpreted as associational rather than strictly causal.

## How to Run
1. Open the `.Rproj` file in RStudio.
2. Run `scripts/01_analysis.R`. The script will read the data from the `data/` folder and save outputs to the `output/` folder.
