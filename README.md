# A Multi-Criteria Approach for Optimising the Spatial Planning of EV Charging Infrastructures

**Student:** Yuyue Xia | **Programme:** MSc Urban Spatial Science, UCL Bartlett CASA  
**Supervisors:** Prof. Chen Zhong (primary) · Dr. Changwha Oh (co-supervisor)  
**Module:** CASA0004 Dissertation | **Year:** 2025–2026

---

## Research in One Sentence

This study develops a multi-criteria location-allocation model that integrates spatially heterogeneous demand estimation and UK planning constraints to optimise the deployment of public EV charging infrastructure in Greater London, with explicit spatial equity objectives using the Index of Multiple Deprivation 2019.

---

## Repository Structure

```
Dissertation/
│
├── 00_admin/                        # Project management
│   ├── meeting_notes/               # Supervision meeting notes
│   ├── timeline.md                  # Week-by-week plan (START HERE)
│   ├── research_diagram.md          # Visual overview of methodology
│   └── File_Structure.md            # This file explained
│
├── 01_literature/                   # Literature review
│   ├── papers_pdf/                  # PDF copies of papers
│   ├── reading_notes/               # Notes per paper
│   └── literature_matrix.md        # Gap analysis table
│
├── 02_ai_conversations/             # AI-assisted work logs
│   └── *.md                         # Dated conversation exports
│
├── 03_data/
│   ├── demand/                      # Demand-side public data
│   │   ├── census2021/
│   │   │   ├── TS001_usual_residents_London_LSOA_2021.csv
│   │   │   └── TS045_car_van_availability_London_LSOA_2021.csv
│   │   ├── imd2019/
│   │   │   └── IMD2019_LSOA_England.xlsx
│   │   └── nts/
│   │       └── nts-2024-excel-tables/
│   │
│   ├── supply/                      # Supply-side public data
│   │   ├── osm/                     # OpenStreetMap GLA
│   │   └── os_open_roads/           # OS Open Roads
│   │
│   ├── policy/                      # Policy reference documents
│   │   └── HoC_EV_policy_briefing_2025.pdf
│   │
│   └── restricted/                  # ⛔ NOT on GitHub (.gitignored)
│       ├── gla/                     # Supervisor-provided CSVs
│       │   ├── device_all_uk.csv
│       │   ├── evse_status_gla.csv
│       │   ├── gla_location_evse_join.csv
│       │   └── OpenStreetEV_GLA.csv
│       └── examples/                # ✅ Anonymised 100-row samples (on GitHub)
│           ├── device_all_uk_sample.csv
│           ├── evse_status_gla_sample.csv
│           ├── gla_location_evse_join_sample.csv
│           └── OpenStreetEV_GLA_sample.csv
│
│
├── 04_code/
│   ├── NOTEBOOKS.md                 # Index of all notebooks
│   ├── 00_create_samples.ipynb
│   ├── 01_data_loading.ipynb
│   └── 02_data_cleaning.ipynb       # (and future notebooks)
│
├── 05_processed/                    # Cleaned data outputs from 02_data_cleaning.ipynb
│ 
├── 06_outputs/
│   ├── figures/                     # Maps and charts
│   ├── model_results/               # Optimisation outputs
│   └── tables/                      # Summary statistics
│
├── 07_writing/
│   ├── dissertation_draft.docx      # Main dissertation document
│   └── presentations/               # Supervision slides
│
├── .gitignore
└── README.md                        # This file
```

---

## Data Sources

### Demand-Side (Public)

| Dataset | Source | Geography | Use |
|---|---|---|---|
| TS001 Usual Residents | Nomis / Census 2021 | London LSOA | Population proxy for demand |
| TS045 Car/Van Availability | Nomis / Census 2021 | London LSOA | Vehicle ownership proxy |
| National Travel Survey 2024 | DfT | National (GB) | Trip frequency, Pd and Ppub parameters |
| IMD 2019 | MHCLG | England LSOA | Spatial equity weighting |

### Supply-Side

| Dataset | Source | Format | Use |
|---|---|---|---|
| device_all_uk | Supervisor (restricted) | CSV | Charger device attributes |
| evse_status_gla | Supervisor (restricted) | CSV | Charger status data |
| gla_location_evse_join | Supervisor (restricted) | CSV | Location–charger linkage |
| OpenStreetEV_GLA | Supervisor (restricted) | CSV | GLA charger spatial data |
| OpenStreetMap GLA | Geofabrik | Shapefile | Road network, candidate sites |
| OS Open Roads | OS DataHub | GeoPackage | Road network analysis |

### Policy

| Document | Source | Use |
|---|---|---|
| EV Infrastructure Strategy 2022 | GOV.UK | Policy constraints & targets |
| House of Commons EV Briefing 2025 | Parliament | Current policy context |

> **Note:** Restricted datasets are excluded from this repository via `.gitignore`.  
> Anonymised 100-row example files are available in `03_data/restricted/examples/`.  
> Full data available upon request from supervisors.

---

## Methodology Overview

```
Step 1: Demand Estimation
  Census (TS001, TS045) + NTS 2024 + IMD 2019
  → Spatial proxy Di for EV charging demand at LSOA level
  → Formula: Di = Pi × Ci × Pd × Ppub × IMD_weight_i

Step 2: Candidate Site Generation
  OSM + OS Open Roads → public car parks, on-street locations
  Supply constraints: site eligibility, grid capacity, planning policy

Step 3: Location-Allocation Modelling (PuLP)
  - P-median: minimise total weighted travel distance
  - Set-covering: cover all demand within radius R
  - Maximal covering: maximise covered demand with p facilities

Step 4: Scenario Comparison
  A. Baseline (efficiency only)
  B. Equity-weighted (IMD penalty term)
  C. Coverage-maximising

Step 5: Evaluation
  - Unmet demand · Average travel distance
  - IMD deprivation index overlap
  - Sensitivity analysis
```

---

## Key Papers

| Paper | Contribution to This Study |
|---|---|
| Vazifeh et al. (2019) *Transportation Research Part A* | Set-covering formulation; time robustness justifies static demand approach |
| He et al. (2022) *Transportation Research Part A* | Contextualised planning constraints; capacitated LA model with CPLEX |
| Lu et al. (2026) *Sustainable Cities and Society* | Land-use heterogeneity justifies LSOA-level demand; utilisation mechanism |

**Core Research Gap:** None of the above papers incorporates spatial equity objectives. This study integrates IMD 2019 deprivation weighting into the location-allocation framework.

---

## Ethics

CASA LREC application submitted and approved.  
Reference: **CASA LREC-2026-4063** | Low-risk secondary data analysis.

---

> Restricted data files must be obtained separately from supervisors and placed in `03_data/restricted/gla/`.
