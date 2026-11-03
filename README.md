# Replicating Card (1990): The Mariel Boatlift and the Miami Labor Market

Reproducible replication and extension of David Card's 1990 study, *"The Impact of the Mariel Boatlift on the Miami Labor Market,"* using difference-in-differences and synthetic control on CPS microdata.

## Overview

In 1980, roughly 125,000 Cubans arrived in Miami within a few months, raising the city's labor force by about 7%. Card (1990) used this episode as a natural experiment to ask how a sudden, large inflow of immigrants affects the wages and employment of existing workers. Comparing Miami with a set of other cities before and after the boatlift, he found little to no measurable effect on wages or unemployment, including among less-skilled and Black workers.

This repository:

1. **Replicates** Card's original difference-in-differences design as closely as the available data allow.
2. **Extends** it with modern methods: event-study specifications and synthetic control.
3. **Tests robustness** to the sample, skill-group, and comparison-city choices raised in the later literature (Borjas 2017; Peri and Yasenov 2019).

Everything is scripted end to end, so every table and figure can be regenerated from raw data.

## Research design

| Component | Description |
|---|---|
| Treated unit | Miami, FL |
| Comparison cities | Atlanta, Houston, Los Angeles, Tampa-St. Petersburg (Card's original set) |
| Event | Mariel boatlift, April to October 1980 |
| Outcomes | Log hourly wages, unemployment rate, by race/ethnicity and skill group |
| Data | Current Population Survey (CPS), 1979 to 1985 |
| Estimators | Difference-in-differences, event study, synthetic control |

The baseline specification compares the change in outcomes in Miami before versus after 1980 with the change in the comparison cities over the same period.

## Repository structure

```
.
├── README.md
├── data/
│   ├── raw/             # Not tracked. Downloaded IPUMS extract goes here.
│   └── processed/       # Cleaned analysis files (generated)
├── code/
│   ├── 01_clean.py      # Build analysis sample from raw CPS extract
│   ├── 02_replicate.py  # Card (1990) difference-in-differences tables
│   ├── 03_eventstudy.py # Event-study estimates and plots
│   ├── 04_synth.py      # Synthetic control
│   └── 05_robustness.py # Alternative samples, skill groups, comparison cities
├── output/
│   ├── tables/
│   └── figures/
├── environment.yml      # Pinned environment
└── Makefile             # One-command rebuild
```

The code is organized as a numbered pipeline. Each script reads only from `data/` and writes only to `data/processed/` or `output/`.

## Data access

The raw data are **not included** in this repository. CPS microdata are available for free from [IPUMS CPS](https://cps.ipums.org/), which requires registration and does not permit redistribution of extracts.

To build the extract:

1. Create an account at <https://cps.ipums.org/> and start a new extract.
2. Select the CPS samples for **1979 to 1985** (basic monthly and/or ASEC, depending on the specification; see `code/01_clean.py` for the exact selection).
3. Select the variables used in the analysis: `YEAR`, `MONTH`, `METAREA` (or `METRO`/`MSA` identifier), `AGE`, `SEX`, `RACE`, `HISPAN`, `EDUC`, `EMPSTAT`, `LABFORCE`, `EARNWEEK`, `HOURWAGE`, `UHRSWORKORG`, and the survey weights (`WTSUPP`/`EARNWT`).
4. Download the extract and save it as `data/raw/cps_extract.csv.gz` (or update the path in `code/config.py`).

Note that metropolitan area identifiers are limited in the early CPS files, so the sample of identifiable cities and the exact match to Card's original sample may differ. Any departures from the original are documented in `docs/differences_from_card.md`.

## Getting started

```bash
# Clone the repository
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# Create the environment
conda env create -f environment.yml
conda activate mariel

# Place the IPUMS extract in data/raw/, then rebuild everything
make all
```

`make all` runs the full pipeline and regenerates every table and figure in `output/`. Individual steps can be run with `make clean_data`, `make replicate`, `make eventstudy`, `make synth`, and `make robustness`.

## Methods

**Difference-in-differences.** The baseline estimate is the coefficient on `Miami × Post` in a regression of the outcome on city and year effects, with CPS survey weights and standard errors that account for the small number of treated clusters (see the robustness script for wild-cluster bootstrap and randomization-inference alternatives).

**Event study.** Leads and lags of the Miami indicator are used to check pre-trends and to show the dynamics of the effect, with 1979 as the reference year.

**Synthetic control.** A weighted combination of comparison cities is constructed to match Miami's pre-1980 outcome path. Following Peri and Yasenov (2019), the donor pool is a broader set of metropolitan areas, and inference is based on placebo-in-space tests.

## Related literature

- Card, D. (1990). The Impact of the Mariel Boatlift on the Miami Labor Market. *Industrial and Labor Relations Review*, 43(2), 245-257.
- Borjas, G. J. (2017). The Wage Impact of the Marielitos: A Reappraisal. *Industrial and Labor Relations Review*, 70(5), 1077-1110.
- Peri, G., and Yasenov, V. (2019). The Labor Market Effects of a Refugee Wave: Synthetic Control Method Meets the Mariel Boatlift. *Journal of Human Resources*, 54(2), 267-309.
- Clemens, M. A., and Hunt, J. (2019). The Labor Market Effects of Refugee Waves: Reconciling Conflicting Results. *ILR Review*, 72(4), 818-857.

## Reproducibility

- Environment is pinned in `environment.yml`.
- Random seeds are set in `code/config.py` for all bootstrap and permutation procedures.
- The `Makefile` documents the dependency order between scripts.

## Contributing

Issues and pull requests are welcome, especially corrections to the sample construction or suggestions for additional robustness checks.

## License

Code is released under the MIT License. Data are subject to the IPUMS CPS terms of use and are not redistributed here.

## Citation

If you use this repository, please cite the original study and this replication:

```
Card, D. (1990). The Impact of the Mariel Boatlift on the Miami Labor Market.
Industrial and Labor Relations Review, 43(2), 245-257.
```
