# Chapter 6 — Moving Averages & Algorithmic Trading

We close the course with a fun, applied topic that ties together everything you've built — Python data structures, pandas DataFrames, model-evaluation metrics, and time series. We start with the **moving-average model** (a forecast so simple it isn't even really a "model"), learn to **evaluate** it honestly (bias, MAE, MAPE, RMSE), and then do something slick: turn two moving averages into an **algorithmic trading bot** that decides when to buy and sell a stock. Along the way you'll get a refresher on lists vs. NumPy arrays, building DataFrames, and the `.rolling()` window function. *(This material adapts the first two chapters of* Data Science for Supply Chain Forecast *— light on theory, rich in code — plus a public algorithmic-trading notebook.)*

## 6.1 Overview: where we're going

Two notebooks this week. The first — **Moving Averages & Evaluating a Model** — refreshes basic Python data structures (lists, NumPy arrays), builds DataFrames from them, then creates a **moving-average model from scratch** and judges its quality both **visually** and with **error metrics** (mean error / bias, mean absolute error). The second notebook does something slick: **algorithmic trading**. We imagine we want to invest in the stock market and build an algorithm to trade *for us* to maximize profit. We fit **two** moving-average models, and where the two moving-average prices **intersect** becomes our signal to **buy or sell**. *(None of this is financial advice — it's an exercise to practice the moving-average model and have some fun.)*

## 6.2 Python data refresher: lists, arrays & DataFrames

Two useful data structures. **Lists** can hold mixed types and be sliced/indexed by position — but `TS + TS2` on two lists **concatenates** them (appends one onto the other), which is *not* math. Convert to **NumPy arrays** with `np.array(...)` and the same `+` does **element-wise addition** — the output you wanted. Rule of thumb: if you're doing math on numeric data, use NumPy arrays; a lot of behind-the-scenes formatting happens for you. NumPy functions like `np.mean([...])` even coerce a list on the fly. Slicing recap (Python counts from 0): `my_list[1]` is the second element; `my_list[1:3]` starts at index 1 up to *but not including* 3; negative indices chop from the end.

Lists and arrays are "technically correct" but the data is just floating around — it's nicer in an Excel-like layout, which is exactly a **pandas DataFrame**. Build one from a **list of lists** (each inner list becomes a **row**, giving *wide* form — each row a time series) or, more conveniently, from a **dictionary** (`pd.DataFrame(d)` — each key becomes a column) in one line. **Rename columns** by overwriting `df.columns = [...]` or with `df.rename(columns={0: "iPhones", 1: "iPads"})` (remember to assign the result back). Subset a column with `df["day1"]` (bracket-and-quote style — it "lights up red," and mixing single/double quotes looks sloppy, so pick one). `df.loc[...]` is label-location; `df.iloc[0, 0]` is integer-location. Two columns need a **list inside the brackets**: `df[["a", "b"]]`. Often you want *long* form (each time series a **column**), so **`.T`** transposes the frame — the first row becomes the first column.

## 6.3 Building the moving-average model from scratch

Take a one-dimensional demand series `D` and forecast each day from the mean of the previous **`n`** observations (a **window** of size `n`). If today is Wednesday and `n = 3`, we average Sunday/Monday/Tuesday to predict Wednesday. Code a function `def moving_average(D, extra_periods, n):` — the three arguments are the **data**, **extra periods** (empty future rows to forecast into, appended with `np.append(D, [np.nan] * extra_periods)`), and the **window** `n`. Make a forecast array filled with `np.nan`, then **loop** `for t in range(n, cols+1): F[t] = D[t-n:t].mean()` — the mean of the previous `n` steps. Where there's no data left, keep repeating the last mean. Return a DataFrame of **demand, forecast, and error** (`error = actual − forecast`). On a 23-day series with `n=3`, the orange forecast line **threads the needle** between the peaks and valleys — a desirable property when the signal is fairly stable. Customize the plot (figure size, `ylim`, dashed vs. solid lines) for a cleaner presentation.

You don't *have* to write your own — pandas has it built in: **`df["demand"].rolling(3).mean()`** gives the identical result (retrieve the past `n` observations, take their mean). But coding it yourself reveals what it does — and you might be asked to write your own function on the job or on an assignment. (How would you have found `.rolling()`? Google "pandas moving average" → GeeksforGeeks / Stack Overflow.)

## 6.4 Evaluating the model: bias, MAE, MAPE, RMSE

A moving average isn't a machine-learning or even really a *statistical* model — it's a data-processing technique with a single knob (the look-back window). Still, you need a **quantitative** way to describe its fit. Define **error = forecast − actual** per time step (subscript *t*). Four core regression metrics (the LaTeX in the notebook — `$...$`, subscripts, the summation symbol — renders those crisp formulas):

- **Bias** = mean of the errors. Does the model **systematically over- or under-estimate**? Zero = unbiased; positive = overestimates; negative = underestimates. Positive and negative errors **cancel**.
- **MAE** (mean absolute error) = mean of the **absolute** errors — no cancellation, so a −8.6 counts like a +8.6. "On average, off by X units," in the problem's original units, but **direction-free**.
- **MAPE** (mean absolute percentage error) = divide each error by the **actual** → "off by 6% on average." But it **blows up when actuals are zero or small**, so only use it when zeros aren't common.
- **RMSE** (root mean squared error) = square the errors, average, take the root — so it's **penalized by outliers** (a big miss is squared and dominates).

No single metric is enough; **present several** — together they tell the story. Compute them fast with **scikit-learn**: `from sklearn.metrics import mean_absolute_error, mean_squared_error` (and `squared=False` for RMSE) — but **drop the NaN warm-up rows first** (`df.dropna(inplace=True)`) or it errors. Illustration: two near-identical forecasts differing only on one outlier point (actual 15, predicted 8 vs. 9) — the better forecast improves **RMSE by 6.8%** but **MAE by only 3.6%**, because RMSE is far more sensitive to that single outlier. That's *why* you report both.

## 6.5 From forecasting to trading: the two-moving-average signal

Now the fun reframe. Instead of forecasting demand, forecast a **stock price** — and use two moving averages as a **momentum** indicator. Pull ~5 years of daily prices (the notebook uses **GLD**, the gold ETF) via `pandas_datareader` → Yahoo Finance (upgrade pandas + `pandas_datareader` first). Keep the **closing** price and compute a **short** (9-day) and a **long** (21-day) rolling mean.

The critical, easy-to-miss step: **shift the moving averages down by one** (`.shift(1)`). The 9-day average as computed *includes today's close*, but on the morning you'd trade you don't yet know today's close — using it **leaks future data**. Shifting by one period fixes the leakage so your signal only uses information you'd actually have. Plot the price with both averages: where the **9-day crosses the 21-day**, momentum is turning. The **strategy**: when the **9-day < 21-day**, momentum is downward → **sell**; when the **9-day > 21-day** (21-day < 9-day), there's an upswing → **buy**, and ride the wave. The 9 and 21 aren't sacred — you should **tune** them (even a brute-force `for`-loop grid search), e.g. try 30/200 or 30/100.

## 6.6 Building the trading bot & the buy-and-hold benchmark

Turn the crossover into code. Make a **`signal`** column: **+1** where the 9-day > 21-day (bought in), **−1** where 9-day < 21-day (out), carrying the value forward otherwise; drop the NaN warm-up. Signals start after day 21. To measure money, work with **log returns**: `np.log(close).diff()` — the natural log makes returns **add** instead of multiply (log rules), and up-days are positive, down-days negative. The **buy-and-hold** benchmark is just the `return` column: invest \$1 on day 1 and hold for five years. The **algorithm's** return is the slickest line in the notebook — **`system = signal.shift * return`**: when `signal` is +1 you're in and earn the day's return; when it's −1 you're out and earn nothing (even if the price rose). Summing, exponentiating, and subtracting 1 converts log returns back to a **percentage growth**; cumulative return uses `np.cumsum` of the logs (adding logs = multiplying), then `np.exp` for the actual portfolio value. The **difference of the signal** marks trade events: **+2 = buy**, **−2 = sell**; plotting them gives ~34 green (buy) and 34 red (sell) triangles over the full history.

The honest punchline: for GLD, **buy-and-hold grew ~32–38%**, while the **9/21 trading bot grew only ~8.9%**. Being *in* the market beats being out, but this simple bot **didn't beat buy-and-hold** — the market tends to rise over time, so "just hold" is a tough benchmark. Tuning the windows (30/100, 30/200) closes some of the gap but doesn't clearly win. A good validation exercise: **fit on data through, say, June 2022, then test on the held-out period** and see if the strategy still performs. *(Again — not investment advice; it's a Python exercise, safely inside Colab with no real money on the line.)*

## 6.7 Where the course goes next (a teaser)

Completing BDA1, you are **data-competent**: you can read, clean, merge, and describe data; resample and forecast time series; understand where statistical numbers (margins of error, confidence intervals) come from; and display data visually with points, lines, and polygons — even joining CSVs to polygons. The natural next question is *what do I do with all this data?* That's **modeling** — predicting an outcome from your features. A quick teaser: pull CT-data tables (educational attainment, per-capita income, home sales), reshape long→wide, `merge` them into one clean frame, and fit a **decision-tree regressor** in a few lines to predict **per-capita income** — reaching an **R² ≈ 0.92** (off by ~\$3,000 on average). Trees are transparent: you can **plot the tree** to see its decisions and read off **feature importance** (college attainment is a strong driver). But easy-to-run code hides real theory — *knowing what to do when* is the whole point of the next course, **BDA2**. You've built the foundation; now go make something useful with it.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Quick overview of moving averages and algorithmic trading
:class: note dropdown
- Two notebooks: build a **moving-average model** + evaluate it, then apply it to **algorithmic trading**.
- Evaluate visually **and** with error metrics (mean error/bias, MAE).
- Trading idea: fit **two** moving averages; their **crossover** signals **buy or sell**.
- Not financial advice — an exercise to practice the model and have fun.
:::

:::{admonition} Moving Average Model Pt. 1 (Python data refresher)
:class: note dropdown
- **Lists** allow mixed types; `list + list` **concatenates**. **NumPy arrays** do **element-wise math**.
- Build a **DataFrame** from a **list of lists** (wide) or a **dictionary** (`pd.DataFrame(d)`).
- Subset with `df["col"]`, `.loc` (labels), `.iloc` (integers); two columns need a **list of names**.
- **`.T`** transposes wide→long (a row becomes a column); rename via `df.columns=[...]` or `df.rename`.
:::

:::{admonition} Moving Average Model Pt. 2 (building it from scratch)
:class: note dropdown
- `moving_average(D, extra_periods, n)`: forecast each step as the **mean of the previous `n`** observations.
- Append `np.nan` future rows; loop `F[t] = D[t-n:t].mean()`; return **demand, forecast, error**.
- A window-3 forecast **threads the needle** between peaks and valleys on a stable signal.
- pandas shortcut: **`df["col"].rolling(3).mean()`** gives the identical result.
:::

:::{admonition} Moving Average Model Pt. 3 (evaluation metrics)
:class: note dropdown
- **Bias** = mean error (over/under-estimate; cancels sign). **MAE** = mean absolute error (direction-free, real units).
- **MAPE** = % error (blows up on zero/small actuals). **RMSE** squares errors → **penalized by outliers**.
- **No single metric is enough** — present several together.
- Use `sklearn.metrics` (`squared=False` for RMSE); **drop NaN warm-up rows** first.
:::

:::{admonition} Momentum Trading Pt. 1 (the two-average signal)
:class: note dropdown
- Pull ~5 yr daily prices (GLD) via `pandas_datareader` → Yahoo; keep **close**; make **9-day** & **21-day** means.
- **`.shift(1)`** the averages to avoid **data leakage** (today's close isn't known when you trade).
- **9-day < 21-day → sell** (downward momentum); **9-day > 21-day → buy** (upswing).
- Windows 9/21 aren't fixed — **tune** them (grid search; try 30/100, 30/200).
:::

:::{admonition} Momentum Trading Algorithm Pt. 2 (the bot vs. buy-and-hold)
:class: note dropdown
- **`signal`** = +1 (in) / −1 (out) from the crossover; work with **log returns** (`np.log(close).diff()`).
- **`system = signal × return`** — earn the day's return only when bought in; the slickest line in the notebook.
- Signal **diff**: +2 = buy, −2 = sell (~34 buys/34 sells over the history).
- Result: **buy-and-hold ~32–38%** beat the **9/21 bot (~8.9%)** — being in the market helps, but simple crossover didn't beat holding.
:::

:::{admonition} Wrapping up: a teaser for BDA2 (modeling)
:class: note dropdown
- After BDA1 you're **data-competent**: read/clean/merge/describe, resample & forecast, stats, and mapping.
- Next is **modeling** — predict an outcome from features (e.g., per-capita income from home sales + education).
- A **decision-tree regressor** in a few lines can reach **R² ≈ 0.92**; plot the **tree** and **feature importance**.
- Easy code hides real theory — **knowing what to do when** is the focus of **BDA2**.
:::
