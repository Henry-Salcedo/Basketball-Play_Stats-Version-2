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
- Reads `players_stats_by_season_full_details.csv` using `numpy.genfromtxt`, then filters rows to retain only **NBA regular season** records based on the league/stage column.
- Identifies and returns the **player with the most regular-season appearances** by counting unique seasons per player name.
- Designed as the single entry point for data access so that all other classes receive clean, pre-filtered arrays rather than raw file I/O.

### `PlayerSeasonAnalyzer`
- Responsible for the **per-player scipy analyses**: regression, integration, and interpolation.
- Accepts the filtered dataset and the name of the most-seasoned player, then extracts that player's season-by-season 3P% values.
- Uses `scipy.stats.linregress` to fit a linear trend line to the player's 3P% over their career years.
- Uses `scipy.integrate.quad` to integrate the fit line and calculate the area-under-the-curve average 3P%, which is then compared to the simple arithmetic average of actual season values.
- Uses `scipy.interpolate.interp1d` to estimate the missing 3P% values for the **2002-2003** and **2015-2016** seasons via interpolation between adjacent known seasons.

### `ColumnStatsAnalyzer`
- Responsible for **dataset-wide statistical analysis** of the FGM and FGA columns.
- Accepts the full filtered FGM and FGA arrays and computes descriptive statistics (mean, variance, skew, kurtosis) using `scipy.stats.describe` or individual `scipy.stats` functions.
- Performs a **paired t-test** (`scipy.stats.ttest_rel`) between FGM and FGA to test whether there is a statistically significant difference between the two related columns.
- Performs **individual one-sample t-tests** (`scipy.stats.ttest_1samp`) on FGM and FGA separately, then prints a comparison of all t-test results.

---

## c) Class Attributes and Methods

### `NBADataLoader`

**Attributes:**
- `filepath` *(str)* – Path to the CSV file (`players_stats_by_season_full_details.csv`).
- `raw_data` *(numpy.ndarray)* – Full numeric data array loaded from the CSV, before filtering.
- `metadata` *(numpy.ndarray)* – String columns (player name, season, league/stage) loaded separately for filtering and labeling.
- `nba_data` *(numpy.ndarray)* – Numeric data filtered to NBA regular season rows only.
- `nba_metadata` *(numpy.ndarray)* – String metadata filtered to NBA regular season rows only.
- `most_seasoned_player` *(str)* – Name of the player who appeared in the greatest number of distinct regular seasons.

**Methods:**
- `__init__(self, filepath)` – Constructor that stores the file path and initializes all data attributes to `None`.
- `load(self)` – Reads the CSV using `numpy.genfromtxt`, populates `raw_data` and `metadata`, then calls `filter_nba_regular_season()` and `find_most_seasoned_player()`.
- `filter_nba_regular_season(self)` – Filters `raw_data` and `metadata` to rows where the league/stage column equals `"NBA"` (regular season), storing results in `nba_data` and `nba_metadata`.
- `find_most_seasoned_player(self)` – Counts distinct seasons per player in `nba_metadata`, stores the player with the highest count in `most_seasoned_player`, and returns their name.

---

### `PlayerSeasonAnalyzer`

**Attributes:**
- `player_name` *(str)* – The name of the player being analyzed.
- `seasons` *(numpy.ndarray)* – Array of season start years (integers) for each season the player played.
- `tp_accuracy` *(numpy.ndarray)* – Computed 3P% (three-point field goals made / attempted) for each season.
- `regression_slope` *(float)* – Slope of the linear regression line fit to `tp_accuracy` vs. `seasons`.
- `regression_intercept` *(float)* – Intercept of the linear regression line.
- `regression_r_value` *(float)* – Pearson correlation coefficient (r) from the regression.
- `integrated_avg_3p` *(float)* – Average 3P% computed by integrating the regression line and dividing by the season span.
- `actual_avg_3p` *(float)* – Simple arithmetic average of `tp_accuracy` across all seasons played.
- `interpolated_values` *(dict)* – Dictionary mapping missing season years (`2002`, `2015`) to their interpolated 3P% estimates.

**Methods:**
- `__init__(self, player_name, nba_data, nba_metadata)` – Constructor that stores the player name and dataset, then calls `extract_player_seasons()` to populate `seasons` and `tp_accuracy`.
- `extract_player_seasons(self)` – Filters `nba_metadata` and `nba_data` to rows matching `player_name`, computes 3P% per season using `numpy.divide` with zero-division safety, and populates `seasons` and `tp_accuracy`.
- `run_linear_regression(self)` – Calls `scipy.stats.linregress(seasons, tp_accuracy)` and stores `regression_slope`, `regression_intercept`, and `regression_r_value`.
- `integrate_fit_line(self)` – Defines the regression line as a lambda and calls `scipy.integrate.quad` over the season range; divides the result by `(latest_season - earliest_season)` to get `integrated_avg_3p`, where the denominator is the length of the interval in years (e.g., seasons spanning 1996–2016 use a divisor of 20). Prints a comparison against `actual_avg_3p`.
- `interpolate_missing_seasons(self, missing_years)` – Builds a `scipy.interpolate.interp1d` interpolator from known seasons and 3P% values, evaluates it at each year in `missing_years`, and stores results in `interpolated_values`.
- `display_results(self)` – Prints the regression equation, the integrated average vs. actual average 3P%, and the interpolated values for the missing seasons.

---

### `ColumnStatsAnalyzer`

**Attributes:**
- `fgm` *(numpy.ndarray)* – Array of Field Goals Made values for all NBA regular season rows.
- `fga` *(numpy.ndarray)* – Array of Field Goals Attempted values for all NBA regular season rows.
- `fgm_stats` *(scipy.stats.DescribeResult)* – Descriptive statistics (mean, variance, skew, kurtosis) for FGM.
- `fga_stats` *(scipy.stats.DescribeResult)* – Descriptive statistics (mean, variance, skew, kurtosis) for FGA.
- `paired_ttest_result` *(tuple)* – Result of `scipy.stats.ttest_rel(fgm, fga)` as `(t_statistic, p_value)`.
- `fgm_ttest_result` *(tuple)* – Result of `scipy.stats.ttest_1samp(fgm, popmean)` as `(t_statistic, p_value)`.
- `fga_ttest_result` *(tuple)* – Result of `scipy.stats.ttest_1samp(fga, popmean)` as `(t_statistic, p_value)`.

**Methods:**
- `__init__(self, fgm, fga)` – Constructor that stores the FGM and FGA arrays and initializes all result attributes to `None`.
- `compute_descriptive_stats(self)` – Calls `scipy.stats.describe` on both `fgm` and `fga`, storing the results in `fgm_stats` and `fga_stats`, and prints a side-by-side comparison of mean, variance, skew, and kurtosis.
- `run_paired_ttest(self)` – Calls `scipy.stats.ttest_rel(fgm, fga)` and stores the result in `paired_ttest_result`.
- `run_individual_ttests(self)` – Calls `scipy.stats.ttest_1samp` on `fgm` and `fga` individually (using the overall mean of each column as the population mean) and stores results in `fgm_ttest_result` and `fga_ttest_result`.
- `display_ttest_comparison(self)` – Prints the t-statistic and p-value for the paired t-test and both individual t-tests, with an explanation of how the results compare.

---

## d) Limitations

- **Dataset dependency** – The program requires a specific CSV file (`players_stats_by_season_full_details.csv`) to be present at the expected path. If the file is missing or has a different column structure, the program will fail to load data.
- **Regular season filter** – The filter relies on a specific string value in the league/stage column. If the dataset uses inconsistent labeling (e.g., `"NBA "` with a trailing space), rows may be incorrectly excluded.
- **Three-point accuracy division by zero** – Seasons in which the most-seasoned player attempted zero three-pointers will produce a 3P% of `0.0`, which can skew the regression and integration results.
- **Interpolation range** – `scipy.interpolate.interp1d` can only interpolate between known data points; it cannot extrapolate beyond the earliest or latest season. Missing seasons at the edges of the player's career cannot be estimated.
- **Paired t-test assumption** – `scipy.stats.ttest_rel` assumes the FGM and FGA arrays are the same length and observations are paired. Rows with missing values that were filled with `0` may violate this assumption.
- **One-sample t-test population mean** – The individual t-tests use the sample mean of each column as the population mean. Testing a sample against its own mean always produces a t-statistic of exactly 0 and a p-value of 1.0, making the result mathematically degenerate. A meaningful population mean (e.g., a league-wide historical average) should be used instead.
- **Console-only output** – All results are printed to the terminal. There is no file export, plotting, or GUI output in the current implementation.
