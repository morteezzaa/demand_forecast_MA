# EU-26 Electricity Demand Model

Projects electricity demand for heat pumps, the rest of the buildings sector, and data centers
across 26 European countries (EU-27 minus Malta and Cyprus, plus Norway) from 2025 to 2060,
under four scenarios (KKO, KVE, VWE, VNI - see below). Built for the master's thesis, in
cooperation with EnBW.

## Pipeline

Six notebooks, run in this order:

1. **`data_preparation.ipynb`** - reads the raw Eurostat/SSP/other input files and builds one
   country-year panel (population, GDP, HDD/CDD, industry value added, energy intensity, past
   electricity demand by sector, ...). Output: `Data - raw/prepared_data/panel_transformed.csv`.
2. **`floor_area_panel.ipynb`** - reconstructs useful floor area and its age structure per
   country for 2010-2024 from the EU Building Stock Observatory snapshot and observed Eurostat
   building permits, then projects 2025-2060 by scenario (renovation, demolition, demand-driven
   new construction). Output: `Data - raw/prepared_data/floor_panel_2010_2060.csv`.
3. **`heat_pump_ols.ipynb`** - PanelOLS model of heat demand, converted to heat-pump electricity
   demand via a heat-pump-share projection and a COP model. Outputs:
   `Data - raw/prepared_data/hist_hp_heat_demand.csv`, `Results/hp_demand_proj.csv`.
4. **`buildings_rest_ols.ipynb`** - same PanelOLS approach for the remainder of buildings
   electricity demand (total buildings demand minus heat pumps minus data centers). Needs step
   3's output first. Output: `Results/building_demand_proj.csv`.
5. **`praesentation_grafiken.ipynb`** - takes the result files from steps 2-4 (plus the data
   center numbers, see below) and produces the figures used in the slide deck, exported to
   `figures/`.

**`run_full_pipeline.ipynb`** is the orchestrator: it holds every tunable scenario parameter
(renovation rate, demolition rate, HP adoption curve, COP, ...) in one `PARAMS` dict, writes
them out to `scenario_config.py`, and then runs notebooks 1-4 (plus 5) in order with a fresh
kernel each. Each notebook still runs fine standalone - it just reads whatever is currently in
`scenario_config.py`. Use the orchestrator when you want to change an assumption; use the
individual notebooks when you're working on one model in isolation.

Two inputs are **not** produced by this pipeline and have to exist beforehand:

- `Results/data_centers_demand.xlsx` - data center demand, modeled separately.
- `Data - raw/chdd_data/output_csv/hdd_cdd_scenarios_2025_2060.csv` - the four SSP-linked
  HDD/CDD paths, produced by `chdd_calculation_scenarios.ipynb` in `Data - raw/chdd_data/`.

`run_full_pipeline.ipynb` checks both files exist before running anything.

## Scenarios

Four scenarios run through the whole pipeline, each tied to one SSP climate/socioeconomic
storyline:

| Code | SSP | Storyline |
|------|-----|-----------|
| KKO | SSP1 | sustainability - Renovation Wave target met, fast HP adoption |
| KVE | SSP2 | middle of the road - partial policy success |
| VWE | SSP5 | fossil-fuelled development - high growth, slower retrofit |
| VNI | SSP3 | regional rivalry - status quo, slowest transition |

Population, GDP and floor-area demand are the same across scenarios; renovation rate, HP
adoption speed and the climate path (HDD/CDD) are what actually differ between them.

## Folder structure

```
Data - raw/            raw input files + prepared_data/ (intermediate panels)
Results/               model outputs (hp_demand_proj.csv, building_demand_proj.csv, ...)
figures/               PNGs exported by praesentation_grafiken.ipynb
flags/                 cached country flag icons (downloaded once, used in the bump chart)
pipeline_runs/         timestamped executed-notebook copies from run_full_pipeline.ipynb
Obsolet/                old notebooks and drafts, superseded - not part of the pipeline
scenario_config.py     auto-generated from run_full_pipeline.ipynb's PARAMS - do not edit by hand
```

## Running it

Open `run_full_pipeline.ipynb`, adjust `PARAMS` (or pick one of the presets in section 3) and
run all cells. To rerun a single model with the current parameters, just open that notebook and
run it directly - it will pick up `scenario_config.py` as it currently is.
