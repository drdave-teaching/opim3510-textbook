# Chapter 5 — Advanced Wrangling & Model Interpretation

Toy datasets are clean and single-file; **real-world data is messy and scattered**. This chapter builds the wrangling skills to **join multiple datasets** on a **primary key**, handle **mismatched keys** and **inner vs. outer joins**, and **impute** the missing values that would otherwise crash a model. Then we add the missing piece of the ML story — **model interpretation** via **permutation importance** — so you can say *which features actually drive the prediction*.

## 5.1 Working with real-world data

The clean datasets so far (breast cancer, Boston/California housing) came as a single complete file. Reality gives you **multiple CSVs to join** — e.g. a **happiness** dataset (~149 countries) plus a **quality-of-life** dataset (~80 countries). You join them on a shared key, then **impute** the columns that are missing for the countries lacking quality-of-life data — because any missing value **crashes a machine-learning model** (it can't do math on a blank). Two imputation tools: **mean** and **median**.

## 5.2 Identifying the primary key

First, use the **right reader**: `pd.read_excel` for `.xls`, `pd.read_csv` for `.csv` (even a spreadsheet-looking CSV). A **primary key** is a **unique identifier** for each row — here, `country`/`country_name`. Unlike Access, you don't flag it; you just *recognize* it. Matching is **exact** — a stray capital letter, underscore, or space breaks the join. Use **`.isin()`** to check overlap: `df_happy[df_happy["country_name"].isin(df_qol["country"])]` returns the matching rows (77 of 83 match). Flip it to check the other direction, and prepend **`~`** for the *non*-matches — revealing the 6 countries in QOL that aren't in happiness (Oman, Qatar, Taiwan, Puerto Rico, Bosnia and Herzegovina, Hong Kong). The dataset with **more rows** (richer) becomes the **left** table; the subset is the **right** table.

## 5.3 Inner vs. outer joins

Some of the 6 mismatches are genuine (Oman, Qatar, Puerto Rico — just absent), so **drop** them; others are **key mismatches** to fix: `Hong Kong` → `Hong Kong SAR of China`, `Bosnia and Herzegovina` (case difference), `Taiwan` → `Taiwan Province of China`. Overwrite a cell with **`np.where(df["country"] == "Taiwan", "Taiwan Province of China", df["country"])`**. Then **`pd.merge`** with a primary key from each side and a **`how`**:
- **Inner join** (`how="inner"`) keeps only **perfect matches** (→ 80 rows, the minimum of the two sets). The merge auto-aligns columns via the key, so no error-prone Excel copy-paste.
- **Outer join** (`how="outer"`) keeps **all left rows** (149) and fills unmatched right columns with **`NaN`**. Here the less-happy countries lack quality-of-life data — which we'll impute (you can't impute the categorical `country`, but numeric columns take the mean/median).

## 5.4 Mean vs. median imputation

After the outer join, `temp2.info()` shows 80 non-null (69 missing) in the QOL columns. Drop the now-duplicate `country` key column (`drop(columns=..., inplace=True)`), leaving all-numeric data. Impute in **one line** rather than column-by-column: **`temp2.fillna(temp2.mean())`** (fills each numeric column's blanks with its mean of the 80 known values) — or **`.median()`**. The choice shifts values slightly (a column mean of 78.33 vs. median 79.25). Crucially, the whole process is **traceable and auditable**: the raw files are untouched, every step is a commented cell, and you **write the cleaned result to a new CSV** (`x.to_csv(path)`) so modeling can start from a fresh notebook — separate cleanup, modeling, and evaluation scripts keep things organized.

## 5.5 Subsetting hail and flash-flood records

A preview of text analytics using a ~61,000-row **storm** dataset (event ID, **`event_type`** = the label, plus a long **episode narrative** and a short **event narrative**). We focus on the **short** narrative — some are blank, so **`dropna`** that column (→ 48,000 rows). A Seaborn **bar plot of `value_counts`** shows the class distribution (thunderstorm wind most common, then floods, hail, high wind, winter weather); keep only categories with **> 2000** records via `groupby`. **Balanced classes** matter for classification (so accuracy is meaningful), so we subset **hail OR flash flood** with a logical `|` (each condition parenthesized): `df[(df["event_type"] == "Hail") | (df["event_type"] == "Flash Flood")]` (~7,000 rows, ~3,800 flash flood + ~3,700 hail). Finally **`reset_index(drop=True)`** — `drop=True` prevents pandas from re-inserting the old index as a column. Text models won't *read* the sentence left-to-right (that needs a recurrent neural network); they'll ask **does this word appear** — the subject of the next chapter.

## 5.6 A gentle introduction to permutation importance

Linear regression's best feature is the **star** marking significant predictors. Is there an equivalent for black-box ML models? **Permutation importance.** The idea: ML models are pattern-based, so if you **shuffle one column** of the **test** partition (leaving the trained model and other columns intact) and the score barely changes, that feature **wasn't important**; if the score **collapses**, it was crucial. From scratch: baseline R² = 0.668; shuffle the `lstat` column in place (`sample` + `reset_index`) and R² drops to ~0.46 (rerun for a **distribution** — use the mean); shuffle an unimportant column and R² barely moves (0.668 → 0.63). Two payoffs: **communication** ("this predictor doesn't matter, don't pay for that data") and **parsimony** — drop unimportant features and refit a **simpler** model that performs about the same.

## 5.7 Implementing permutation importance

scikit-learn does it directly: `from sklearn.inspection import permutation_importance`. Declare your fitted model (conventionally **`clf`**), then `permutation_importance(clf, X_test, y_test, n_repeats=10)` — **use the test partition** (unseen data) and reference the **already-fitted** model (no re-fitting). Plot the results as a sorted **box plot** (a bit of raw Matplotlib — define axes, fill the figure, `show()`), updating only the **title** per model. For Boston Housing's linear regression the top features are `lstat`, `dis`, `rm`, `rad`; the **decision tree** and **random forest** rank features differently because each algorithm treats data differently — but **rooms** consistently matters. Then pick a cutoff, **refit on the top features**, and if the reduced model's R² is close to the full model's (e.g. 0.68 vs. 0.70), prefer the **simpler** model (a gut call, not a formal test). That closes the loop: join disparate data, clean and impute it, model it, and **explain which features drive the prediction**.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Working with real-world data is complicated
:class: note dropdown
- Real data comes as **multiple files to join** (e.g. happiness ~149 rows + quality-of-life ~80 rows), not one clean CSV.
- Join on a shared key, then **impute** the columns missing for unmatched rows.
- Any **missing value crashes** a machine-learning model — it can't do math on a blank.
- Two imputation tools: **mean** and **median**.
:::

:::{admonition} Identifying the primary key in different dataframes
:class: note dropdown
- Use **`pd.read_excel`** for `.xls`, **`pd.read_csv`** for `.csv` — the right tool for the file.
- A **primary key** is a unique row identifier; matching is **exact** (case/underscore/space breaks it).
- **`.isin()`** checks overlap; prepend **`~`** for the non-matches.
- The dataset with **more rows** (richer) is the **left** table; the subset is the **right**.
:::

:::{admonition} Inner Joins vs. Outer Joins
:class: note dropdown
- Fix mismatched keys (e.g. `Hong Kong → Hong Kong SAR of China`) by overwriting cells with **`np.where`**; drop genuinely-absent rows.
- **`pd.merge(..., how="inner")`** keeps only perfect matches (the minimum set); the key auto-aligns columns.
- **`how="outer"`** keeps **all left rows** and fills unmatched right columns with **`NaN`**.
- You can't impute the categorical key column — only numeric columns.
:::

:::{admonition} Mean vs. median imputation for missing data
:class: note dropdown
- After the join, `info()` reveals the missing count; drop the duplicate key column, leaving numeric data.
- Impute in one line: **`df.fillna(df.mean())`** or **`.median()`** — the choice shifts values slightly.
- The process is **traceable/auditable** — raw files untouched, every step commented.
- **Write the cleaned result to CSV** so modeling starts from a fresh notebook.
:::

:::{admonition} Subsetting hail and flash flood records
:class: note dropdown
- Storm data: `event_type` (label) + short/long narratives; **`dropna`** the blank short narratives.
- Seaborn **`value_counts`** bar plot shows class distribution; keep frequent categories via `groupby`.
- **Balanced classes** matter for classification — subset **hail OR flash flood** with a parenthesized `|`.
- **`reset_index(drop=True)`** — `drop=True` stops pandas re-inserting the old index as a column.
:::

:::{admonition} A gentle introduction to permutation importance
:class: note dropdown
- Permutation importance is the ML answer to linear regression's "significance star."
- **Shuffle one column of the test partition**; a big score drop = important feature, no change = unimportant.
- Shuffling gives a **distribution** of scores (rerun and use the mean); baseline R² 0.668 → ~0.46 for `lstat`.
- Payoffs: communicate which features matter, and **parsimony** — drop the rest and refit a simpler model.
:::

:::{admonition} Implementation of permutation importance
:class: note dropdown
- `permutation_importance(clf, X_test, y_test, n_repeats=10)` — use the **test** set and the **already-fitted** model.
- Plot a sorted **box plot**, updating only the **title** per model.
- Different algorithms rank features differently (but **rooms** consistently matters on Boston Housing).
- **Refit on the top features**; if R² is close to the full model, prefer the **simpler** one (a gut call).
:::
