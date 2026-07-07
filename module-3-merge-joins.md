# Chapter 3 — Data Wrangling: Merge, Joins, Concat & Queries

Real analysis rarely comes from one tidy file — you combine sources, then **query** them. This chapter covers the two ways to stitch DataFrames together — **`merge`** (join on a **key**, like SQL — inner, left, outer) and **`concat`** (paste rows or columns) — and then **answering business questions with logical statements** (`groupby`, `np.where`, nested conditions), in both Excel and Python. We use two weather datasets — **precipitation** (412 stations) and **temperature** (347 stations) — that don't share all their station-date pairs, so combining them by hand in Excel would be a nightmare.

## 3.1 Merge vs. concat, and the axis convention

Two tools:
- **`merge`** joins on a **primary key** (a common field), keeping/dropping rows by match — inner, outer, or left.
- **`concat`** pastes DataFrames together — **rows** (`axis=0`) or **columns** (`axis=1`).

The **axis convention runs through all of pandas**: **`axis=0` = rows, `axis=1` = columns**. (Memory hook: `df.shape` prints `(rows, columns)`, so element 0 is rows, element 1 is columns.) The precip data has ~151k rows × 29 columns from 412 stations; temperature has ~127k rows × 21 columns from 347 stations — similar shape, different coverage, which is exactly why we need join tools. *(Note: load data with `gdown` (pin v4.6) so scripts run without mounting Drive.)*

## 3.2 Inner join (the most common)

**`pd.merge(left, right, left_on=..., right_on=..., how="inner")`** keeps only rows whose keys **match** in both frames, discarding the rest. Subset one station's precip data (365 rows × 29 cols) and merge it (inner) with the full temperature data: you get **365 rows or fewer** (fewer if some dates don't match) and **47 columns** (29 precip + 18 temperature) — a richer table. Always **check the shape** after any manipulation. Merging on **two keys** (station **and** date) across the full datasets (127k × 151k) yields **123,005 matching rows**. Watch for **suffixes** — duplicated columns get `_x` (left) and `_y` (right). Inner join = perfect data quality, at the cost of dropped rows.

## 3.3 Left join

**`how="left"`** keeps **every row of the left** frame and fills the right side where it matches (**`NaN`** where it doesn't). So `left=climate_temp, right=precip_one_station, how="left"` always returns **127,020 rows** (same as the left), with mostly-missing precip columns — great left-side quality, poor right-side. Flip it (`left=precip_one_station`) and you get **365 rows** — a left join **always has the same row count as the left frame**, unlike an outer join which can grow *beyond* either input.

## 3.4 Outer join

**`how="outer"`** keeps **all rows from both** frames — matches align, and non-matches get **`NaN`**. Combining one station's 365 precip rows with the 127k temperature rows gives **127,020 rows** (or more) with lots of missing precip values in the tail. Inspect with **`.info()`** (non-null counts) and **`.isna().sum()`** (missing per column). To clean up, **`df.fillna(0, inplace=True)`** imputes the blanks (deeper missing-data handling is a BDA2 topic) — after which `.info()` shows complete data. Outer join = maximum information, lower data quality until you impute.

## 3.5 Concat

**`pd.concat`** pastes frames together — think of the old `append`. **Rows** (`axis=0`): stack one below the other (e.g. Meta's 2020 prices CSV atop its 2021 CSV to plot a full history) — concatenating a 365-row frame with itself gives 730 rows (use **`ignore_index=True`** to reset the messy index). **Columns** (`axis=1`): paste side by side — a 29-column frame with itself gives 58 columns, same rows.

The crucial caveat: **`concat` has no key — it's blind copy-paste.** If your data has order (a time component) and the two sources are sorted differently, concat will happily produce a **garbage result you won't notice**. So for anything ordered or keyed, **prefer `merge`** (with `left_on`/`right_on`, or on the index) — it aligns by key. (`join` is similar to `merge`; treat it as optional.)

## 3.6 Business queries with logicals (the Facebook data)

Once data is combined, you **query** it. Using a "Facebook" student dataset (id, gender, hours online, friends), four common tasks — first in **Excel**, then **Python**:

1. **Means per group** — "average hours online for males vs. females?" In Excel, a **pivot table** (gender as rows, mean of hours). In pandas, **`df.groupby("gender").mean()`** (or `.agg({"friends": "mean"})` for one column); `reset_index()` turns the group into a column.
2. **A flag with AND** — "high advertising" if male AND >6 hrs AND >150 friends, else "low." Excel: `IF(AND(...))`. Python: **`np.where((cond1) & (cond2) & (cond3), "high", "low")`** — each condition **parenthesized**, joined with **`&`** (not the word `and`).
3. **A nested condition** — low/medium/high by friend count. Excel: a **nested `IF`**. Python: nested `np.where`, or split into two `np.where`s (or a `fillna`).
4. **A flag with OR** — "high-end user" if >8 hrs **OR** >300 friends. Excel: `IF(OR(...))`. Python: **`np.where((cond1) | (cond2), ...)`** with the **`|`** operator.

A real-world Excel gotcha: messy files put titles/notes in the top rows, so **`pd.read_excel(path, skiprows=2)`** (and `read_excel`, not `read_csv`, for `.xlsx`). Python's advantage is **transparency** — everything is laid out and re-runnable, and the recipe transfers to new data with small tweaks, whereas a stray edit in an Excel formula can silently corrupt results.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Intro to Merge, Joins and Concat
:class: note dropdown
- Two ways to combine data: **`merge`** (join on a key) and **`concat`** (paste rows/columns).
- **`axis=0` = rows, `axis=1` = columns** throughout pandas (`df.shape` = (rows, columns)).
- Example data: precipitation (412 stations) + temperature (347 stations) — different coverage.
- Load with **`gdown`** (pin v4.6) to avoid mounting Drive.
:::

:::{admonition} Inner Joins (most common)
:class: note dropdown
- **`pd.merge(..., how="inner")`** keeps only rows whose **keys match** in both frames.
- Merging adds the right frame's columns (365×29 + temp → 365×47); result is "matching rows **or fewer**."
- Merge on **multiple keys** (station + date); duplicated columns get **`_x`/`_y`** suffixes.
- Always **check `.shape`** after merging; inner = best quality, dropped rows.
:::

:::{admonition} Left join
:class: note dropdown
- **`how="left"`** keeps **every left row**, fills the right where matched, **`NaN`** where not.
- Result **always has the same row count as the left** frame.
- Left-side data quality is high; right-side may be mostly missing.
- Unlike an outer join, a left join never grows beyond the left frame.
:::

:::{admonition} Outer Joins
:class: note dropdown
- **`how="outer"`** keeps **all rows from both** frames; non-matches become **`NaN`**.
- Inspect with **`.info()`** (non-null) and **`.isna().sum()`** (missing per column).
- Clean up with **`fillna(0, inplace=True)`** (deeper imputation is a BDA2 topic).
- Maximum information, lower data quality until imputed.
:::

:::{admonition} Concat
:class: note dropdown
- **`pd.concat`** pastes frames: rows (`axis=0`) stack, columns (`axis=1`) go side by side.
- Use **`ignore_index=True`** to reset the index after row-concatenation.
- **No key** — it's blind copy-paste; mismatched ordering yields silent garbage.
- Prefer **`merge`** for anything ordered/keyed; `join` is a similar (optional) alternative.
:::

:::{admonition} Facebook with Excel
:class: note dropdown
- Answer business questions with **logicals**: means per group, flags, nested conditions.
- **Pivot table** for group means; **`IF(AND(...))`** for a multi-condition flag; **nested `IF`** for low/med/high.
- **`IF(OR(...))`** for an either-or flag (e.g. high-end user).
- Excel works, but a stray formula edit can silently corrupt results.
:::

:::{admonition} Facebook with Python
:class: note dropdown
- Same four questions in pandas — more **transparent** and re-runnable than Excel.
- **`groupby(...).mean()`** / `.agg({col: fn})` for group means; `reset_index()` to flatten.
- **`np.where`** with parenthesized conditions joined by **`&`** (AND) / **`|`** (OR); nest for 3-way flags.
- Messy Excel files: **`read_excel(path, skiprows=2)`** (and `read_excel`, not `read_csv`, for `.xlsx`).
:::
