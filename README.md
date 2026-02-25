# Basketball-Play_Stats-Version-2
Project will be focused on the uses of scipy to determine NBA calculations.

---

## a) Purpose of the Project

- This project is the second version of the Basketball Players Stats Analysis program, upgraded to leverage **scipy** for more advanced statistical and mathematical computations.
- The program reads a CSV dataset of NBA player statistics spanning multiple seasons, then filters the data to include **NBA regular season records only**.
- Using the filtered dataset, the program identifies the **player who has appeared in the most regular seasons**, then performs a series of scipy-powered analyses on that player's career statistics.
- Key analyses include:
  - Calculating the player's **three-point accuracy (3P%)** for every season they played.
  - Performing a **linear regression** (`scipy.stats.linregress`) on their 3P% across seasons to produce a line of best fit.
  - **Integrating** the regression line (`scipy.integrate.quad`) over the span of seasons played to compute the average 3P% from the fit, and comparing it to the player's actual average 3P%.
  - Using **interpolation** (`scipy.interpolate`) to estimate the player's 3P% for the **2002-2003** and **2015-2016** seasons, in which they did not participate.
  - Computing **descriptive statistics** (mean, variance, skew, kurtosis) for the **Field Goals Made (FGM)** and **Field Goals Attempted (FGA)** columns across the full dataset using `scipy.stats`, and comparing the two distributions.
  - Performing a **paired (relational) t-test** (`scipy.stats.ttest_rel`) on FGM and FGA, as well as individual **one-sample t-tests** (`scipy.stats.ttest_1samp`) on each column, and comparing the results.
- The goal is to apply scipy's scientific computing capabilities—regression, integration, interpolation, and hypothesis testing—to real NBA data.

---

## b) Class Design and Implementation

The program is organized into three classes, each with a focused responsibility:

### `NBADataLoader`
- Responsible for **loading and filtering** the raw CSV dataset.
- Reads `players_stats_by_season_full_details.csv` using `numpy.genfromtxt` — numeric columns (`FGM`, `FGA`, `3PM`, `3PA`, `FTM`, `FTA`, `PTS`, `MIN`, `GP`, `BLK`, `STL`) are loaded separately from string columns (`Player`, `Season`, `Stage`, `League`).
- Filters rows to keep only those where the `Stage` column equals `"Regular Season"`, producing the NBA regular season subset used by all downstream classes.
- Identifies and returns the **player with the most regular-season appearances** by counting unique `Season` values per `Player` name.
- Designed as the single entry point for data access so that all other classes receive clean, pre-filtered arrays rather than raw file I/O.

### `PlayerSeasonAnalyzer`
- Responsible for the **per-player scipy analyses**: regression, integration, and interpolation.
- Accepts the filtered dataset arrays and the name of the most-seasoned player, then extracts that player's rows to compute `tp_acc = 3PM / 3PA` for each season using `numpy.divide` with safe zero-division handling.
- Uses `scipy.stats.linregress` to fit a linear trend line to `tp_acc` over `seasons` (season start years parsed from the `Season` string, e.g., `"1999 - 2000"` → `1999`).
- Uses `scipy.integrate.quad` to integrate the fit line over `[earliest_season, latest_season]` and divides by the interval length to compute the average 3P% from the fit; compares it to the arithmetic mean of the actual `tp_acc` values.
- Uses `scipy.interpolate.interp1d` to estimate the missing 3P% values for the **2002-2003** (`2002`) and **2015-2016** (`2015`) seasons by interpolating between adjacent known seasons.

### `ColumnStatsAnalyzer`
- Responsible for **dataset-wide statistical analysis** of the `FGM` and `FGA` columns.
- Accepts the full filtered `FGM` and `FGA` arrays extracted from `nba_data` and computes descriptive statistics (mean, variance, skewness, kurtosis) using `scipy.stats.describe`, then prints a side-by-side comparison of the two distributions.
- Performs a **paired (relational) t-test** (`scipy.stats.ttest_rel`) between `FGM` and `FGA` to test whether there is a statistically significant difference between the two related columns.
- Performs **individual one-sample t-tests** (`scipy.stats.ttest_1samp`) on `FGM` and `FGA` separately, then prints a comparison of the paired t-test vs. the individual t-test results.

---

## c) Class Attributes and Methods

### `NBADataLoader`

**Attributes:**
- `filepath` *(str)* – Path to the CSV file (`players_stats_by_season_full_details.csv`).
- `player_stats` *(numpy.ndarray)* – Full numeric data array loaded from the CSV using `numpy.genfromtxt`, before filtering. Numeric columns include `FGM`, `FGA`, `3PM`, `3PA`, `FTM`, `FTA`, `PTS`, `MIN`, `GP`, `BLK`, `STL`.
- `metadata` *(numpy.ndarray)* – String columns (`Player`, `Season`, `Stage`, `League`) loaded separately via `numpy.genfromtxt` with `dtype=str` for filtering and labeling.
- `nba_data` *(numpy.ndarray)* – Subset of `player_stats` filtered to rows where the `Stage` column equals `"Regular Season"` and the `League` column equals `"NBA"`.
- `nba_metadata` *(numpy.ndarray)* – Subset of `metadata` filtered to the same NBA regular season rows.
- `most_seasoned_player` *(str)* – Name of the player who appeared in the greatest number of distinct `Season` values in the filtered dataset.

**Methods:**
- `__init__(self, filepath)` – Constructor that stores the file path and initializes all data attributes to `None`.
- `load(self)` – Reads the CSV using `numpy.genfromtxt`, populates `player_stats` and `metadata`, then calls `filter_nba_regular_season()` and `find_most_seasoned_player()`.
- `filter_nba_regular_season(self)` – Uses boolean indexing on `metadata` to keep only rows where `Stage == "Regular Season"`, storing results in `nba_data` and `nba_metadata`.
- `find_most_seasoned_player(self)` – Counts distinct `Season` values per `Player` in `nba_metadata`, stores the player with the highest count in `most_seasoned_player`, and returns their name.

---

### `PlayerSeasonAnalyzer`

**Attributes:**
- `player_name` *(str)* – The full name of the player being analyzed, matching the `Player` column in the CSV.
- `seasons` *(numpy.ndarray)* – Array of season start years (integers extracted from the `Season` string, e.g., `"1999 - 2000"` → `1999`) for each season the player appeared in.
- `tp_acc` *(numpy.ndarray)* – Computed three-point accuracy (`3PM / 3PA`) for each season, using `numpy.divide` with `out=np.zeros_like(tpm), where=tpa!=0` to safely handle seasons with zero attempts.
- `regression_slope` *(float)* – Slope of the linear regression line returned by `scipy.stats.linregress(seasons, tp_acc)`.
- `regression_intercept` *(float)* – Intercept of the linear regression line.
- `regression_r_value` *(float)* – Pearson correlation coefficient (r) from `scipy.stats.linregress`.
- `integrated_avg_3p` *(float)* – Average 3P% computed by integrating the regression line using `scipy.integrate.quad` and dividing by the interval length `(latest_season - earliest_season)`.
- `actual_avg_3p` *(float)* – Simple arithmetic mean of `tp_acc` across all seasons played, computed with `numpy.mean`.
- `interpolated_values` *(dict)* – Dictionary mapping missing season start years (`2002` for 2002-2003, `2015` for 2015-2016) to their interpolated `3P%` estimates.

**Methods:**
- `__init__(self, player_name, nba_data, nba_metadata)` – Constructor that stores the player name and filtered dataset arrays, then calls `extract_player_seasons()` to populate `seasons` and `tp_acc`.
- `extract_player_seasons(self)` – Filters `nba_metadata` and `nba_data` to rows matching `player_name` in the `Player` column; extracts the `3PM` and `3PA` columns, computes `tp_acc = np.divide(tpm, tpa, out=np.zeros_like(tpm), where=tpa!=0)`, and parses season start years into `seasons`.
- `run_linear_regression(self)` – Calls `scipy.stats.linregress(seasons, tp_acc)` and stores `regression_slope`, `regression_intercept`, and `regression_r_value`.
- `integrate_fit_line(self)` – Defines the regression fit as `lambda x: regression_slope * x + regression_intercept` and calls `scipy.integrate.quad` over `[earliest_season, latest_season]`; divides the result by `(latest_season - earliest_season)` to get `integrated_avg_3p`. Prints a comparison against `actual_avg_3p`.
- `interpolate_missing_seasons(self, missing_years)` – Builds a `scipy.interpolate.interp1d` interpolator from `seasons` and `tp_acc`, then evaluates it at each year in `missing_years` (e.g., `[2002, 2015]`) and stores the results in `interpolated_values`.
- `display_results(self)` – Prints the regression equation (`tp_acc = slope * season + intercept`), the integrated average vs. actual average 3P%, and the interpolated 3P% values for each missing season.

---

### `ColumnStatsAnalyzer`

**Attributes:**
- `fgm` *(numpy.ndarray)* – Array of `FGM` (Field Goals Made) values extracted from the NBA regular season filtered dataset.
- `fga` *(numpy.ndarray)* – Array of `FGA` (Field Goals Attempted) values extracted from the NBA regular season filtered dataset.
- `fgm_stats` *(scipy.stats.DescribeResult)* – Result of `scipy.stats.describe(fgm)`, containing `nobs`, `minmax`, `mean`, `variance`, `skewness`, and `kurtosis` for the `FGM` column.
- `fga_stats` *(scipy.stats.DescribeResult)* – Result of `scipy.stats.describe(fga)`, containing the same descriptive statistics for the `FGA` column.
- `paired_ttest_result` *(scipy.stats.TtestResult)* – Result of `scipy.stats.ttest_rel(fgm, fga)`, containing `statistic` and `pvalue`.
- `fgm_ttest_result` *(scipy.stats.TtestResult)* – Result of `scipy.stats.ttest_1samp(fgm, popmean)`, containing `statistic` and `pvalue` for the individual FGM t-test.
- `fga_ttest_result` *(scipy.stats.TtestResult)* – Result of `scipy.stats.ttest_1samp(fga, popmean)`, containing `statistic` and `pvalue` for the individual FGA t-test.

**Methods:**
- `__init__(self, fgm, fga)` – Constructor that stores the `FGM` and `FGA` arrays and initializes all result attributes to `None`.
- `compute_descriptive_stats(self)` – Calls `scipy.stats.describe` on both `fgm` and `fga`, storing the results in `fgm_stats` and `fga_stats`, and prints a side-by-side comparison of mean, variance, skewness, and kurtosis for the two columns.
- `run_paired_ttest(self)` – Calls `scipy.stats.ttest_rel(fgm, fga)` and stores the result in `paired_ttest_result`.
- `run_individual_ttests(self)` – Calls `scipy.stats.ttest_1samp(fgm, popmean)` and `scipy.stats.ttest_1samp(fga, popmean)` using a chosen population mean and stores the results in `fgm_ttest_result` and `fga_ttest_result`.
- `display_ttest_comparison(self)` – Prints the t-statistic and p-value for the paired t-test (`ttest_rel`) and both individual t-tests (`ttest_1samp`), then explains how the relational t-test results compare to the individual t-tests.

---

## d) Limitations

- **Dataset dependency** – The program requires `players_stats_by_season_full_details.csv` to be present at the expected path with the expected columns (`Player`, `Season`, `Stage`, `FGM`, `FGA`, `3PM`, `3PA`, etc.). If the file is missing or has a different column structure, `numpy.genfromtxt` will fail to load the data correctly.
- **Regular season filter** – The `Stage` column filter relies on the exact string `"Regular Season"`. If the dataset uses inconsistent casing or whitespace (e.g., `"regular season"` or `"Regular Season "` with a trailing space), rows may be incorrectly excluded.
- **Three-point accuracy division by zero** – Seasons in which the most-seasoned player attempted zero three-pointers (`3PA == 0`) produce a `tp_acc` of `0.0`, which can pull down the regression slope and skew the integration result.
- **Interpolation range** – `scipy.interpolate.interp1d` can only estimate values between the earliest and latest known seasons. If either the 2002-2003 or 2015-2016 season falls outside the player's known career range, the interpolation will raise a `ValueError` rather than estimating the value.
- **Paired t-test assumption** – `scipy.stats.ttest_rel` requires the `FGM` and `FGA` arrays to have the same length and for observations to be paired row-by-row. Rows filled with `0` for missing values may violate the independence assumption.
- **One-sample t-test population mean** – The individual `ttest_1samp` tests use the sample mean of each column as the population mean. Testing a sample against its own mean always produces a t-statistic of exactly 0 and a p-value of 1.0, making the result mathematically degenerate. A meaningful population mean (e.g., a league-wide historical average) should be used instead.
- **Console-only output** – All results are printed to the terminal. There is no file export, plotting, or GUI output in the current implementation.
