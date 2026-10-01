# Titanic Passenger Data: Cleaning & Quality Audit

A documented, tested data cleaning workflow that turns the raw Kaggle Titanic training file into a validated, analysis-ready dataset.

Everything lives in one Jupyter notebook: a data quality audit, reusable cleaning functions, validation checks, unit tests, and a record of every cleaning decision with its reasoning.

## Results at a glance

| | Raw | Cleaned |
|---|---|---|
| Rows | 891 | 891 |
| Columns | 12 | 15 |
| Missing values (excluding `Cabin`) | 179 | 0 |
| Exact duplicate rows | 0 | 0 |

- **Missing values:** `Age` (177), `Cabin` (687) and `Embarked` (2) were each handled with a different, documented strategy.
- **Duplicates:** checked for exact duplicate rows, repeated passenger IDs and rows identical apart from the ID. None were found, and the checks run on every execution.
- **Validation:** the output is checked automatically (no unexpected gaps, valid value ranges, unique IDs, consistent derived columns) and the run stops with a clear error if a rule is broken.
- **Tests:** 11 unit tests cover each cleaning rule, duplicate handling, input validation and the audit function.

## What was cleaned and why

| Issue | Decision | Reason |
|---|---|---|
| `Age` missing (177 rows) | Filled with the median age of the passenger's class and sex group; added an `age_imputed` flag | Median is robust to skew, and age differs clearly by class. The flag lets later analysis treat filled values separately. |
| `Cabin` missing (687 rows, 77%) | Kept as missing; added `has_cabin` and `deck` | Too much is missing to fill honestly. A text placeholder would also hide the gap from `isna()` and SQL `NULL` logic. |
| `Embarked` missing (2 rows) | Filled with the most common port, then `S/C/Q` expanded to port names | Only 2 rows, so the effect of the guess is negligible. |
| `Sex` | Normalised to `Male` / `Female` | Consistent labels; unexpected values raise an error. |
| `Survived` | Stored as `int8` | Stays numeric, so `.mean()` gives the survival rate directly. |
| `Pclass` | Ordered category 1 < 2 < 3 | Encodes that classes have an order. |
| `Fare == 0` (15 rows) | Left unchanged and logged | Could be crew, staff or errors; the data cannot tell which. |
| Column names | Converted to `snake_case` | Consistent and SQL-friendly. |
| Duplicates | Drop exact duplicates; fail on conflicting IDs | Safe automatic fix where it is unambiguous. |

The 210 passengers who share a ticket number with someone else are **not** treated as duplicates, because families and groups travelled on one ticket.

## Output columns

The cleaned file `train_cleaned.csv` has 15 columns:

| Column | Description |
|---|---|
| `passenger_id` | Unique identifier |
| `survived` | 0 = no, 1 = yes |
| `passenger_class` | 1, 2 or 3 (ordered) |
| `name` | Passenger name |
| `sex` | `Male` / `Female` |
| `age` | Age in years (missing values filled) |
| `siblings_spouses` | Siblings or spouses aboard |
| `parents_children` | Parents or children aboard |
| `ticket` | Ticket number |
| `fare` | Fare paid |
| `cabin` | Cabin number (missing kept as missing) |
| `embarked` | Port: Southampton, Cherbourg or Queenstown |
| `age_imputed` | `True` if the age was filled in |
| `has_cabin` | `True` if a cabin was recorded |
| `deck` | First letter of the cabin, or `Unknown` |

## Repository contents

| File | Purpose |
|---|---|
| `titanic_cleaning.ipynb` | The full project: audit, cleaning functions, validation, charts, tests |
| `train_cleaned.csv` | Cleaned output produced by the notebook |
| `train.csv` | Raw input (see *Data source* below) |

## How to run

1. Install the requirements:

   ```bash
   pip install pandas numpy matplotlib jupyter
   pip install duckdb   # optional, only for the SQL example
   ```

2. Put `train.csv` in the same folder as the notebook.
3. Open the notebook and choose **Restart & Run All**.

The notebook writes `train_cleaned.csv` next to itself. It was developed with Python 3 and pandas 3.0.

## Notebook outline

1. Setup
2. Data dictionary
3. Data quality audit (missing values and duplicates)
4. Cleaning functions
5. Cleaning decisions
6. Run the pipeline
7. Validation: before vs after
8. Example insights from the cleaned data
9. Export
10. Unit tests
11. Limitations and next steps

## Limitations and next steps

- **Imputed ages are estimates.** Group medians are reasonable for cleaning, but a model-based approach (for example using the title in `name`) could recover more detail. Use `age_imputed` to test how sensitive any results are.
- **Zero fares were not changed.** Resolving them needs domain knowledge or an outside source.
- **`deck` is only reliable for passengers with a recorded cabin** (about 23% of rows).
- **Possible extensions:** extract passenger titles from `name`, bin ages and fares for modelling, and add a modelling notebook that uses `age_imputed` and `has_cabin` as features.

## Data source

The Titanic dataset comes from the [Kaggle Titanic competition](https://www.kaggle.com/competitions/titanic). Download `train.csv` from there and review Kaggle's terms before redistributing the data.

## Author

[Your name] · [Your Upwork profile or portfolio link] · [Contact]
