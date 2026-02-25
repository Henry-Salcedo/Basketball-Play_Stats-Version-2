# Basketball-Play_Stats-Scipy_Practice

Project will be focused on the uses of scipy to determine NBA calculations for the player with the most amount of reguluar Seaons played, and additional statistical tests done on the Field Goals Made (FGM) column and the Field Goals Attempted (FGA) column.

- This project is of Basketball Players Stats Analysis program, upgraded to leverage **scipy** for more advanced statistical and mathematical computations.
- The program reads a CSV dataset of NBA player statistics spanning multiple seasons, then filters the data to include **NBA regular season records only**.
- Using the filtered dataset, the program identifies the **player who has appeared in the most regular seasons**, then performs a series of scipy-powered analyses on that player's career statistics.

- Key analyses include:
  - Calculating the player's **three-point accuracy (3P%)** for every season they played.
  - Performing a **linear regression** (`scipy.stats.linregress`) on their 3P% (ThreePointAccuracy) across regular seasons to then produce a line of best fit via the mathmatical equation.
  - Using **interpolation** (`scipy.interpolate`) to estimate the player's 3P% for the **2002-2003** and **2015-2016** seasons, in which they did not participate (calculated both linear, cubic, and quad).

  - Computing **descriptive statistics** (mean, variance, skew, kurtosis) for the **Field Goals Made (FGM)** and **Field Goals Attempted (FGA)** columns across the full dataset using `scipy.stats`.
  - Performing a **paired (relational) t-test** (`scipy.stats.ttest_rel`) on FGM and FGA, as well as individual **one-sample t-tests** (`scipy.stats.ttest_1samp`) on each column, and reviewed each of the results.

- The goal is to apply scipy's scientific computing capabilities—regression, integration, interpolation, and hypothesis testing—to real NBA data.

---

## b) Class Design and Implementation

The program is organized into three main classes, each with a focused responsibility:

### `NBADataLoader`
- Responsible for **loading and filtering** the raw CSV dataset.
- Reads `players_stats_by_season_full_details.csv` using `pd.read_csv()`, then filters rows to retain only **NBA regular season** records based on the league/stage column.
- Identifies and returns the **player with the most regular-season appearances** by counting unique seasons per player name.

### `PlayerSeasonAnalyzer`
- Responsible for the **solo-player scipy analyses**: regression, integration, and interpolation.
- Accepts the filtered dataset and the name of the most-seasoned player, then extracts that player's season-by-season 3P% values.
- Uses `scipy.stats.linregress` to fit a linear trend line to the player's 3P% over their career years.
- Uses the formula `slope * x + intercept` to integrate the fit line and calculate the area-under-the-curve average 3P%, which is then compared to the simple arithmetic average of actual season values.
- Uses `scipy.interpolate.interp1d` for linear to estimate the missing 3P% values for the **2002-2003** and **2015-2016** seasons via interpolation between adjacent known seasons. Left both cubic and quadratic calculations in cases I or any users would like to review additonal results pertaining to these versions. 

### `ColumnStatsAnalyzer`
- Responsible for **dataset-wide statistical analysis** of the FGM and FGA columns.
- Accepts the full filtered FGM and FGA arrays and computes descriptive statistics (mean, variance, skew, kurtosis) using individual `scipy.stats` functions.
- Performs a **paired t-test** (`scipy.stats.ttest_rel`) between FGM and FGA to test whether there is a statistically significant difference between the two related columns.
- Performs **individual one-sample t-tests** (`scipy.stats.ttest_1samp`) on FGM and FGA separately, then prints a comparison of all t-test results.

---

## c) Class Attributes and Methods

### `NBADataLoader`

**Attributes:**
- Path to the CSV file (`players_stats_by_season_full_details.csv`), uses computers path way, may need to change for your files if needed.
- Full numeric data array loaded from the CSV using `pandas.read_csv`, before filtering. File contains 34 columns.
- String columns (`Player`, `Season`, `Stage`, `FGM`, `FGA`) are the focus throught the project.
– Subset of `player_stats` filtered to rows where the `Stage` column equals `"Regular Season"`.
- Name of the player who appeared in the greatest number of distinct `Season` values in the filtered dataset.

**Methods:**

- Reads the CSV using `pd.read_csv`.
- Uses boolean indexing on dataset (`sport`) to keep only rows where `Stage == "Regular Season"`, storing results in `sport_reg`.
- Uses `groupby` values of `(Player)[Season]` in `most_seasons`, stores the player with the highest count in `vince`, then returns vinces data for uses in statistical analysis.

---

### `ColumnStatsAnalyzer`

**Attributes:**
- Array of `FGM` (Field Goals Made) values extracted from the NBA regular season `sport_reg` filtered dataset, that was then used on several calculation.
- Array of `FGA` (Field Goals Attempted) values extracted from the NBA regular season `sport_reg` filtered dataset, that was then used on several calculation.
- Result of the calculations, containing `mean`, `variance`, `skewness`, and `kurtosis` for the `FGM` column.
- Result of the calculations, containing `mean`, `variance`, `skewness`, and `kurtosis` for the `FGA` column.
- Result of `scipy.stats.ttest_rel(FGM, FGA)`, containing `T_statistic` and `p-value`.
- Result of `scipy.stats.ttest_rel(FGM, FGA)`, containing `T_statistic` and `p-value` for the individual FGM t-test.
-Result of `scipy.stats.ttest_rel(FGM, FGA)`, containing `T_statistic` and `p-value` for the individual FGA t-test.

**Methods:**
- Constructors that stores the `FGM` and `FGA` calculations.
-Calls 4 calculations on both `fgm` and `fga`, storing the results in `fgm_mean`, `fgm_variance`, `fgm_skew`, and `fgm_kurtosis`. Then the same naming convetion for `fga` was used, and prints a side-by-side comparison of mean, variance, skewness, and kurtosis for the two columns.
- Calls `scipy.stats.ttest_rel(fgm, fga)` and stores the result in `T_statistic`, and `p_value`.
- Calls `scipy.stats.ttest_1samp(fgm, popmean)` and `scipy.stats.ttest_1samp(fga, popmean)` using the means of each columns population mean and stores the results in `t_statistic_fgm` and `t_statistic_fga`.
- Then prints the t-statistic and p-value for the paired t-test (`ttest_rel`) and both individual t-tests (`ttest_1samp`) once they were calculated, then explains how the relational t-test, and the individual t-tests results were returning.

---

## d) Limitations

- The program requires `players_stats_by_season_full_details.csv` to be present at the expected file path with the expected columns (`Player`, `Season`, `Stage`, `FGM`, `FGA`, `3PM`, `3PA`, etc.). If the file is missing, diffrent locate on other operating systems or has a different column structure, `pd.read_csv` will fail to load the data correctly.
- `scipy.stats.ttest_rel` requires the `FGM` and `FGA` to have the same length and for observations to be paired row-by-row. Rows filled with `0` for missing values could violate independence assumptions.
- Within the program when calculating for `..._mean`, `..._variance`, `..._skew`, and `..._kurtosis` (for both `fgm` and `fga`) we instead calculated the seprately for each section instead of just using the `describe` function which could have done them all.
-  Due to limited experience with using t-test within python, lack of review reguarding the math/values uses could be problomatic surrounding the end products.
-  Also limited the project to uses as little other librarys as possible to understand the deeper uses of the `scipy` library.
-  With an introduction to `scipy` being my main understanding, few times relied on self calculating or sticking to equation when functions exsisted that could aliviate this (ex: `describe`).
- All results are printed to the terminal. There is no file export, plotting, or GUI output in the current implementation.
