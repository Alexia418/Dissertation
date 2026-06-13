# Notebook Index

This file describes the purpose and contents of each Jupyter notebook in `04_code/`.

---

## 00_create_samples.ipynb
Creates anonymised 100-row sample files from restricted GLA datasets for version control purposes.
- Hashes sensitive IDs, jitters coordinates, removes sensitive columns
- Output: `03_data/restricted/examples/`

---

## 01_data_loading.ipynb
Loads all raw datasets and prints `.head()` and `.columns` for inspection.
- `TS001_usual_residents_London_LSOA_2021.csv` — Census 2021 population per LSOA
- `TS045_car_van_availability_London_LSOA_2021.csv` — Car/van ownership per LSOA
- `IMD2019_LSOA_England.xlsx` — Index of Multiple Deprivation 2019
- `OpenStreetEV_GLA.csv` — EV charger locations (restricted)
- `gla_location_evse_join.csv` — Location–EVSE join table (restricted)
- `evse_status_gla.csv` — EVSE operational status records (restricted)
- `device_all_uk.csv` — UK-wide EV charging device attributes (restricted)

---

## 02_data_cleaning.ipynb
Cleans and standardises all datasets. Output saved to `05_processed/`.

- **TS001 + TS045**: Filter to London LSOAs; convert float columns to int; compute total vehicles per LSOA and car-ownership rate
- **IMD 2019**: Filter to London LSOAs; handle 2011→2021 LSOA boundary change via lookup table
- **OpenStreetEV GLA**: Validate coordinates within Greater London bounding box; check for duplicate `location_id`
- **EVSE Status**: Aggregate 25 million status records into per-EVSE availability rate
- **Device UK**: Filter to GLA-area devices using join with OpenStreetEV location IDs
