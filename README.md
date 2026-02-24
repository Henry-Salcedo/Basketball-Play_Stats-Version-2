# Basketball-Play_Stats-Version-2
Project will be focused on the uses of scipy to determine NBA calculations.

---

## a) Purpose of the Project

- This project is the second version of the Basketball Players Stats Analysis program, upgraded to leverage **scipy** for more advanced statistical computations.
- The program reads a CSV dataset of NBA player statistics spanning multiple seasons and calculates a variety of performance metrics for each player.
- Key calculations include field goal accuracy (FG%), three-point accuracy (3P%), free throw accuracy (FT%), average points per minute (PPM), overall shooting accuracy, average blocks per game, and average steals per game.
- After computing each metric, the program ranks all players and produces a **Top 100 leaderboard** for every category.
- The goal is to provide a data-driven view of player performance across seasons using efficient, vectorized scientific computing tools (scipy/numpy).

---

## b) Class Design and Implementation

The program is organized into three classes, each with a focused responsibility:

### `PlayerStats`
- Represents a **single player's raw seasonal statistics** loaded directly from the CSV dataset.
- Serves as a lightweight data container (model layer) so the rest of the program works with clean Python objects rather than raw array indices.
- Each instance is created for one player–season combination from the dataset.

### `StatCalculator`
- Responsible for **all statistical computations** using scipy and numpy.
- Accepts a list of `PlayerStats` objects and computes derived metrics (FG%, 3P%, FT%, PPM, overall shooting accuracy, blocks per game, steals per game).
- Uses `scipy.stats` functions (e.g., percentile ranking, descriptive statistics) to enhance the accuracy and depth of the analysis beyond simple division.
- Handles edge cases such as division by zero (e.g., a player with zero field-goal attempts) by returning `0.0` for that metric.

### `LeaderBoard`
- Responsible for **ranking and displaying** the top players for a given computed metric.
- Takes in computed metric arrays and player metadata, sorts them in descending order, and returns the top N players (default 100).
- Provides a formatted print method so each leaderboard is displayed consistently across all categories.

---

## c) Class Attributes and Methods

### `PlayerStats`

**Attributes:**
- `name` *(str)* – The player's full name.
- `season` *(str)* – The season the statistics belong to (e.g., `"2019 - 2020"`).
- `fgm` *(float)* – Field goals made.
- `fga` *(float)* – Field goals attempted.
- `tpm` *(float)* – Three-point field goals made.
- `tpa` *(float)* – Three-point field goals attempted.
- `ftm` *(float)* – Free throws made.
- `fta` *(float)* – Free throws attempted.
- `pts` *(float)* – Total points scored.
- `min_played` *(float)* – Total minutes played.
- `blk` *(float)* – Total blocks.
- `stl` *(float)* – Total steals.
- `gp` *(int)* – Games played.

**Methods:**
- `__init__(self, name, season, fgm, fga, tpm, tpa, ftm, fta, pts, min_played, blk, stl, gp)` – Constructor that initializes all raw statistical attributes for the player.
- `__repr__(self)` – Returns a readable string representation of the player and their season (e.g., `"James Harden | 2018 - 2019"`).

---

### `StatCalculator`

**Attributes:**
- `players` *(list[PlayerStats])* – The list of all `PlayerStats` objects to compute metrics for.
- `fg_acc` *(numpy.ndarray)* – Computed field goal accuracy for each player.
- `tp_acc` *(numpy.ndarray)* – Computed three-point accuracy for each player.
- `ft_acc` *(numpy.ndarray)* – Computed free throw accuracy for each player.
- `ppm` *(numpy.ndarray)* – Computed average points per minute for each player.
- `overall_acc` *(numpy.ndarray)* – Computed overall shooting accuracy (combined FG, 3P, FT) for each player.
- `blk_per_game` *(numpy.ndarray)* – Computed average blocks per game for each player.
- `stl_per_game` *(numpy.ndarray)* – Computed average steals per game for each player.

**Methods:**
- `__init__(self, players)` – Constructor that accepts a list of `PlayerStats` objects and initializes all metric arrays to `None`.
- `calculate_all(self)` – Calls every individual compute method in sequence, populating all metric arrays.
- `calculate_fg_accuracy(self)` – Computes `fgm / fga` for each player using `numpy.divide` with safe zero-division handling; stores result in `fg_acc`.
- `calculate_3p_accuracy(self)` – Computes `tpm / tpa` for each player; stores result in `tp_acc`.
- `calculate_ft_accuracy(self)` – Computes `ftm / fta` for each player; stores result in `ft_acc`.
- `calculate_ppm(self)` – Computes `pts / min_played` for each player; stores result in `ppm`.
- `calculate_overall_accuracy(self)` – Uses `scipy.stats` to compute a weighted overall shooting accuracy combining FG%, 3P%, and FT%; stores result in `overall_acc`.
- `calculate_blk_per_game(self)` – Computes `blk / gp` for each player; stores result in `blk_per_game`.
- `calculate_stl_per_game(self)` – Computes `stl / gp` for each player; stores result in `stl_per_game`.
- `get_percentile(self, metric_array, percentile)` – Uses `scipy.stats.scoreatpercentile` to return the value at a given percentile for any computed metric array.

---

### `LeaderBoard`

**Attributes:**
- `metric` *(numpy.ndarray)* – The computed metric array to rank players by.
- `players` *(list[PlayerStats])* – The corresponding list of `PlayerStats` objects for labeling.
- `label` *(str)* – The display name of the metric (e.g., `"FG%"`, `"PPM"`).
- `top_n` *(int)* – The number of top players to include in the leaderboard (default `100`).
- `ranked_indices` *(numpy.ndarray)* – Indices that sort `metric` in descending order, computed during ranking.

**Methods:**
- `__init__(self, metric, players, label, top_n=100)` – Constructor that sets up all attributes and initializes `ranked_indices` to `None`.
- `rank(self)` – Uses `numpy.argsort` in descending order to populate `ranked_indices` with the top `top_n` positions.
- `display(self)` – Prints a formatted leaderboard table showing rank, player name, season, and metric value for the top `top_n` players.
- `get_top_players(self)` – Returns a list of `(rank, PlayerStats, metric_value)` tuples for the top `top_n` players without printing.

---

## d) Limitations

- **Dataset dependency** – The program requires a specific CSV file (`players_stats_by_season_full_details.csv`) to be present at a hard-coded or user-specified path. If the file is missing or has a different column structure, the program will fail to load data.
- **Missing data handling** – Players with missing or zero values for denominators (attempts, games played, minutes) will receive a computed value of `0.0` for those metrics rather than being excluded from rankings, which may artificially inflate or deflate leaderboard positions.
- **Single-season granularity** – Statistics are tracked per player per season. Career-aggregate rankings are not computed; a player who played 20 seasons will appear multiple times across leaderboards.
- **No weight adjustment for sample size** – A player who appeared in only one game with a perfect shooting night could rank higher than a player with a season-long sample. No minimum games-played or attempts threshold is enforced.
- **Scipy version compatibility** – Some `scipy.stats` functions (e.g., `scoreatpercentile`) are deprecated in newer releases. The implementation targets scipy ≥ 1.7; older or newer versions may require minor adjustments.
- **Console-only output** – All results are printed to the terminal. There is no file export, plotting, or GUI output in the current implementation.
