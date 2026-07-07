# Chapter 2 — pandas, Data Wrangling & EDA

This is where the rubber hits the road. We load real spreadsheet data with **pandas**, inspect its shape and types, compute **statistics**, and **slice and dice** rows and columns by position and condition. Then we lay out a reusable **EDA framework** and work it end-to-end on the **California housing** dataset — mounting Drive, engineering a flag variable, grouping, plotting (box, histogram, KDE, scatter), reading a correlation heatmap, and finishing with a one-line **pandas-profiling** report. The habit to build: approach *any* new dataset with the same recipe.

## 2.1 Reading data with pandas

Three libraries do the heavy lifting: **`pandas` (as `pd`)** for wrangling/stats/viz, **`numpy` (as `np`)** for numeric functions, and **`pylab`** (`from pylab import *`) — a friendlier interface to Matplotlib that lets you call `boxplot`, `hist`, `scatter` without the `plt.` ceremony. Read a CSV with **`pd.read_csv(path)`** and — by universal convention — assign it to **`df`** (never `data` or `my_data`). Preview with **`df.head(10)`**. Note the **index** on the left starts at **0** (Python counts from zero), useful later for row selection and merges. To keep examples manageable, `df = df.head(10)` subsets ten rows.

The single most useful first command is **`df.info()`**: in one line it reports the row count, the columns, their **dtypes**, and **non-null counts** (so you spot missing data instantly). Here everything is `float64` with no nulls — a clean, all-numeric dataset. `df.shape` gives `(rows, columns)` (always that order); `df.columns` lists names; `df.dtypes` gives types. Notice `.shape`/`.columns`/`.dtypes` are **attributes** (no parentheses) while `.info()`/`.head()` are **functions** (parentheses) — a distinction you internalize with practice.

## 2.2 Summary statistics

Statistics describe **center** (mean, median) and **spread** (standard deviation, min/max, percentiles). **`df.describe()`** computes count/mean/std/min/25%/50%/75%/max for every **numeric** column in one line (it ignores strings). For individual values, use either **NumPy** (`np.mean(df["housing_median_age"])`) or **pandas** (`df["housing_median_age"].mean()`) — same result, pick what reads best; swap `mean`→`median`→`std` freely. **Percentiles**: the 25th means 25% of data lies *below* that point; get them with **`df["col"].quantile([0.25, 0.5, 0.75])`** (values on 0–1, not 0–100), and min/max are just the 0 and 1 quantiles. These quartiles power the **box plot** — `boxplot(df["col"])` plus a `ylabel`, a `title`, and a trailing **`show()`** (which suppresses Python's stray text output). A single DataFrame column is a **Series**.

## 2.3 Selecting rows and columns

Wrangling is selecting rows/columns by **position**, **name**, or **condition**:
- **Rows by position** (slice): `df[0:3]` returns rows 0, 1, 2 — the **start is included, the end is not**.
- **Rows by condition**: `df[df["housing_median_age"] > 22]` — a logical that implicitly keeps rows where `True`.
- **Columns by name**: `df[["a", "b"]]` (double brackets for a list of columns).
- **`.loc`** selects by **label/name** (ranges are **inclusive** on both ends); **`.iloc`** selects by **integer position** (end **exclusive**). Use `:` alone to mean "all."
- **Combined & compound**: `df.iloc[1:4, 2:6]`, and combine conditions with **`&`** (AND, more restrictive) and **`|`** (OR, less restrictive), each condition parenthesized.

The payoff: chain a condition into a statistic to answer a business question in two lines — e.g. the 90th-percentile median income for neighborhoods where `population < 1000` **and** `housing_median_age > 20`.

## 2.4 A general EDA framework

Dr. Dave's recipe for any new dataset: (1) **read** it and import packages; (2) inspect **shape, columns, dtypes, and missing values** (or just `df.info()`); (3) **recode / flag** variables to analyze by group (e.g. low- vs. high-population); (4) compute **statistics** and **`groupby`** aggregates (like an Excel pivot table) plus `describe`; (5) make **univariate plots** — box plot (shows the IQR), histogram (bin-size sensitive), and the **kernel density plot** (a smooth histogram, Dr. Dave's favorite, great for overlaying groups); (6) make **bivariate plots** — scatter plots of *y as a function of x* (target on the y-axis), optionally colored by group; (7) examine **correlation** (mainly with the target — in ML, correlated predictors matter less since the model picks the useful one); and (8) the king one-liner, **pandas-profiling**, which auto-generates a full HTML EDA report.

## 2.5 Live demo: California housing

Working the CA housing data from Drive:
- **Mount Drive** (`from google.colab import drive; drive.mount(...)`, authorize, paste the code), navigate to your file's path, and `pd.read_csv("<path>")` — copy the path by right-clicking the file (remember the quotation marks). You can even embed a **house image** with `![caption](url)` to make the notebook presentation-ready.
- **Inspect**: `df.shape` → (17000, 9), `df.columns`, `df.dtypes` (all `float64`), and check missing data with **`df.isna().sum()`** (per-column null counts) — or just `df.info()`.
- **Feature-engineer a flag**: `df["flag_population"] = np.where(df["population"] > 1167, 1, 0)` — a new column, 1 if the condition holds else 0.
- **Group**: `df["median_house_value"].groupby(df["flag_population"]).mean()` — house values run a bit higher in low-population areas here; swap `mean` for `max`/`min`/`quantile`.
- **Plot**: `boxplot`, `hist(..., bins=20)`, and — via **Seaborn** — `sns.kdeplot(df["col"])` and KDE/scatter **by group** with `hue=`. Seaborn's `hue` even accepts a **numeric** column for a smooth color gradient.
- **Correlation**: `df.corr()` gives the pairwise matrix (1's on the diagonal; e.g. total_rooms↔total_bedrooms ≈ 0.92), and `sns.heatmap(corr_matrix)` renders it — focus on the row for **median_house_value** (median income is the strongest driver).
- **One-liner**: `from pandas_profiling import ProfileReport; ProfileReport(df)` builds the full report (you may need to update pandas — a real-world "read the error, self-help" moment). It auto-drops highly correlated columns and shows per-variable stats and plots.

## 2.6 Accessing a file on your Drive

The general pattern for your own data: **drag the file into a Drive folder**, then in a notebook `import pandas as pd`, **mount** the drive, navigate to the file, right-click → **Copy path**, and read it: `pd.read_csv("<path>")` (or `pd.read_excel(...)` for Excel). "Updating your path" just means swapping the middle of that string to point at your file. Next chapter: the machine-learning recipe.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Pandas dataframes and dtypes
:class: note dropdown
- Import **`pandas as pd`**, **`numpy as np`**, and `from pylab import *`; read with **`pd.read_csv`** and name it **`df`**.
- **`df.info()`** is the best one-liner: rows, columns, **dtypes**, and non-null counts (spot missing data).
- `df.shape` = `(rows, columns)`; `df.columns`, `df.dtypes` are **attributes** (no parentheses).
- The **index** and all positions **count from 0**.
:::

:::{admonition} Pandas dataframes and statistics
:class: note dropdown
- **`df.describe()`** gives count/mean/std/min/quartiles/max for numeric columns (ignores strings).
- Get single stats via NumPy (`np.mean(df["col"])`) or pandas (`df["col"].mean()`) — same result.
- **Percentiles** (`df["col"].quantile([...])`) are on **0–1**; min/max are the 0 and 1 quantiles.
- Quartiles power the **box plot**; add `ylabel`, `title`, and a trailing **`show()`**; a single column is a **Series**.
:::

:::{admonition} Selecting rows and columns
:class: note dropdown
- Slice rows by position `df[0:3]` — **start included, end excluded**; filter by condition `df[df["col"] > 22]`.
- Select columns with a list `df[["a","b"]]`; **`.loc`** is by name (inclusive), **`.iloc`** by position (end exclusive).
- Combine conditions with **`&`** (AND) and **`|`** (OR), each parenthesized.
- Chain a condition into a statistic to answer a business question in two lines.
:::

:::{admonition} A General EDA Framework
:class: note dropdown
- Recipe: read → shape/dtypes/missing → **recode/flag** → stats & **groupby** → univariate & bivariate plots → correlation → one-liner.
- Univariate: box plot (IQR), histogram (bin-sensitive), **KDE** (smooth, great by group); bivariate: **scatter** (y ~ x).
- In ML, correlated predictors matter less — focus correlation on the **target**.
- **`groupby`** is a pivot table; **pandas-profiling** auto-generates the whole EDA.
:::

:::{admonition} CA housing EDA — Pt 1 (setup)
:class: note dropdown
- Import `pandas`/`numpy`/`pylab`; **PyLab** is the friendly Matplotlib interface (call `boxplot`/`scatter` directly).
- **Mount Google Drive**, authorize, and paste the code to access your files.
- Read from a Drive path with `pd.read_csv("<path>")` (mind the quotation marks); preview with `df.head(10)`.
- Embed a `![caption](url)` image to make the notebook presentation-ready; comment most lines.
:::

:::{admonition} CA housing EDA — Pt 2 (shape, types, missing)
:class: note dropdown
- `type(df)` confirms a **pandas DataFrame**; `.shape` → (17000, 9), `.columns`, `.dtypes` (all `float64`).
- Check missing data with **`df.isna().sum()`** (per-column counts) — here, none.
- `df.info()` does shape + types + non-null in one line.
- Feature-engineer a **flag** with `np.where(df["population"] > 1167, 1, 0)`.
:::

:::{admonition} CA housing EDA — Pt 3 (statistics & groupby)
:class: note dropdown
- Compute mean/median/std with NumPy or pandas; median left of the mean suggests **right skew**.
- **Percentiles** via `df["col"].quantile([...])`; min/max are the 0/1 quantiles.
- **`groupby`** the flag column to get per-group aggregates (mean/max/min/quantile) — a pivot table.
- `describe` includes the 0/1 flag because pandas reads it as numeric (harmless).
:::

:::{admonition} CA housing EDA — Pt 4 (univariate plots)
:class: note dropdown
- Make plots with PyLab: `boxplot`, `hist(..., bins=N)` — always add labels, a title, and **`show()`**.
- Bin size changes a histogram's shape; the **kernel density plot** (Seaborn `sns.kdeplot`) is smoother.
- Overlay distributions by group with **`hue=`** in Seaborn.
- Three views of one variable: box (center/IQR), histogram (bins), KDE (smooth, favorite).
:::

:::{admonition} CA housing EDA — Pt 5 (bivariate & correlation)
:class: note dropdown
- **`scatter(x, y)`** shows *y as a function of x*; customize `color`/`marker`; color by group with Seaborn `hue=`.
- Seaborn `hue` accepts a **numeric** column for a smooth color gradient.
- **`df.corr()`** gives the pairwise matrix (diagonal = 1); e.g. total_rooms↔total_bedrooms ≈ 0.92.
- **`sns.heatmap(corr_matrix)`** visualizes it — focus on correlations with the **target**.
:::

:::{admonition} CA housing EDA — Pt 6 (pandas profiler)
:class: note dropdown
- **`ProfileReport(df)`** (pandas-profiling) auto-generates a full HTML EDA — "code that writes your code."
- You may need to **update pandas** first — a real-world "read the error, self-help" moment.
- It auto-drops highly correlated columns and shows per-variable stats, distributions, and correlations.
- Practice typing the recipe from memory — copy-paste isn't really coding.
:::

:::{admonition} Accessing a file on your Drive
:class: note dropdown
- **Drag** your CSV/Excel into a Drive folder, then `import pandas`, **mount** the drive, and navigate to it.
- Right-click the file → **Copy path**, and read with `pd.read_csv("<path>")` or `pd.read_excel(...)`.
- "Updating your path" = swapping the middle of that string to point at your file.
- Everything stays in the cloud — no local install needed.
:::
