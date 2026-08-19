# Overview

Dissertation project (UCL CASA) examining how to balance spatial efficiency and deprivation-based equity when allocating additional public EV charging infrastructure across Greater London.The project models existing EV charging provision and utilisation, constructs demand at LSOA level, evaluates the spatial mismatch between supply and demand in relation to deprivation, and uses a p-median facility location model to optimise the allocation of new charging capacity under different equity-weighting (α) and budget (p) scenarios. Results are compared across scenarios to examine the trade-off between overall efficiency and equity across deprivation deciles.

## Repository structure

```
04_code/        Analysis pipeline (Jupyter notebooks), roughly in run order:
  00_new_data_exploration.ipynb   – initial data exploration
  01_data_loading_revised.ipynb   – data loading
  02_data_cleaning_revised.ipynb  – data cleaning
  03_EDA_revised_new.ipynb        – exploratory data analysis
  04_demand_estimation_revised.ipynb – LSOA-level demand constructed
  05_p_median_1.ipynb             – p-median MILP optimisation model
  06_mismatch_revised.ipynb       – supply-demand mismatch & equity analysis
  07_study_area.ipynb             – study area map

03_data/        Raw input data (largely not included — see Data availability)
05_processed/   Cleaned/processed datasets and model outputs
                 (e.g. census, IMD, EVSE registry, ZapMap, demand
                 estimates, p-median assignment results, mismatch-by-
                 decile tables)
06_figures/eda/ Exploratory figures
06_outputs/     Generated result figures and outputs
```

## Data availability

Some of the raw data used in this project — in particular supervisor-provided and other restricted-access datasets — is **not included** in this repository due to data sharing restrictions. Only processed/derived outputs that can be shared are kept under `05_processed/`. To reproduce the full pipeline from raw inputs, these restricted datasets would need to be obtained separately.

## Status

This repository accompanies an MSc dissertation and is not maintained as a general-purpose package.
