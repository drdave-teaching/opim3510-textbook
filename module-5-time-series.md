# Chapter 5 — Time Series & Forecasting: Exponential Smoothing

Time-series forecasting uses the past to predict the future — demand, stock prices, weather. After **moving-average** models (which weight every observation in the window *equally*), we move to **exponential smoothing**, which weights recent observations more and older ones exponentially less, with a tunable knob **α**. We build **simple** and **double** (trend) exponential smoothing **from scratch** in Python, learn to **evaluate** a forecast honestly (MAE, RMSE, residual plots), and finally **grid-search** the best parameters. These are statistical *curve-fitting* models, not machine learning — but simple, effective, and a great foundation.

## 5.1 Introduction to exponential smoothing

You already know how to plot, resample, forward/back-fill, and derive rates of change (`shift`, `diff`) from time series, and how a **moving average** forecasts from past values — where a longer window (200-day vs. 5-day) smooths out the trend, and where **data leakage** (using future/present info) is a danger. **Exponential smoothing** improves on the moving average by letting you **choose a weight** that decays exponentially into the past: recent info matters most, but the past still informs the forecast. Today's focus is the **simple** model plus strong **model-evaluation** skills — reading a time series, fitting the model, and judging its fit with scatter plots, actual-vs-predicted lines, and residuals.

## 5.2 Motivation and initialization

Instead of equal weights, exponentially weight recent values and fade the past — which is often better because **outliers and noise have less impact**. The model: **Fₜ = α·Dₜ₋₁ + (1−α)·Fₜ₋₁** — the forecast blends the **most recent observation** (learning something new) with the **previous forecast** (remembering the past), and α + (1−α) = 1. **α** (the smoothing parameter / learning rate, usually 0–0.8) is the knob: **high α** → very responsive to recent data (but jumpy on noise); **low α** → smooth, slow to notice level changes. Why "exponential"? A window-3 moving average gives each of 3 points ⅓ weight, but exponential smoothing weights *all* history — at α=0.4, yesterday gets 40%, two days ago 24%, three days ago 14%… a smooth **drop-off**. Because there's no observation before the start, **initialize** the first forecast to the first actual (`F₀ = D₀`) — "completely phony, but you have to get the party started."

## 5.3 Building simple exponential smoothing from scratch

Because the initial forecast equals the actual, **don't trust the earliest forecasts** (data leakage) — focus later in the series. Code a function `simple_exp_smoothing(D, extra_periods, alpha)`: convert `D` to a **NumPy array** (fast, list-like), get its length, append `np.nan` for the forecast horizon, make a forecast array with **`np.full(..., np.nan)`**, set `F[0] = D[0]`, then loop **`F[t] = α·D[t-1] + (1-α)·F[t-1]`**. Future periods just **repeat the last value** (these models don't extrapolate). Return a DataFrame of demand, forecast, and error (actual − predicted). Tune **α**: 0.4–0.5 looks sharp; too low is clunky (a long warm-up); **α ≈ 0.99 collapses to a moving average of window 1**. On a 21-point series with 4 forecast periods, the output has 25 rows. (Stats-models/scikit-learn have canned versions, but coding your own reveals what it does — and impresses in a whiteboard interview.)

## 5.4 Wrapping up simple exponential smoothing

Comparing **α = 0.1** (smooth, orange) vs. **α = 0.8** (bumpy) shows the trade-off; both need a warm-up. Why "exponential"? Expand the recursion — Fₜ₋₁ itself expands into Dₜ₋₂ and Fₜ₋₂, and so on — so **the forecast is a function of *all* past observations and forecasts**, with α controlling the emphasis. As α → 1, you use only the most recent value (a window-1 moving average). **Limitations:** future forecasts are a **flat constant** (the last fitted value repeated) — the model **can't capture trend** (need **double** exponential smoothing), **can't capture seasonality** (need **triple** — e.g. winter-coat sales), and **can't use external drivers** like price/marketing (that's regression). The author calls simple ES possibly "too simple," but as we've seen, **simple is often effective** — it's how you use and evaluate it that adds value.

## 5.5 Introduction to double exponential smoothing

Add **trend**. Beyond α (level), introduce **β** (trend), so the **forecast = level + trend**. Level looks familiar: **level = α·Dₜ + (1−α)·(prev level + prev trend)**; **trend = β·(current − previous change) + (1−β)·(prev trend)**. For a horizon λ, **Fₜ₊λ = level + λ·trend** — so unlike simple ES, double ES **extrapolates a slope into the future**. Initialize **level = D₀** and **trend = D₁ − D₀** (so the first forecast is perfect, then degrades). Tuning α vs. β changes how the fit tracks level vs. trend. The catch: it just keeps **adding a constant trend forever**, so it loses value over long horizons.

## 5.6 Double exponential smoothing from scratch

In Python, keep three NaN-filled arrays — **level `A`**, **trend `B`**, **forecast `F`** — each length + extra periods. Initialize `A[0] = D[0]`, `B[0] = D[1] − D[0]`. Loop: forecast `F[t] = A[t-1] + B[t-1]`, then **update level** `A[t] = α·D[t-1] + (1-α)·(A[t-1] + B[t-1])` and **update trend** `B[t] = β·(A[t] − A[t-1]) + (1-β)·B[t-1]`. The future forecast carries the last trend forward (e.g. subtracting 5.24 or 0.55 each step) — which **breaks** over a long horizon (try `extra_periods=15` and watch it run away). So it's good for **short horizons**. Next: find the best α, β automatically — and evaluate with metrics, scatter, line, and histogram plots throughout.

## 5.7 Evaluating how your model did

The output DataFrame has actual, forecast, and **error = actual − predicted** (a **residual**: overestimates read *negative*, underestimates *positive* — Dr. Dave prefers **predicted − actual** so the sign matches the direction; either is fine as long as you **tell your audience** which). Metrics coded from scratch: **MAE** (mean absolute error — average miss in the problem's units, e.g. "off by 3.06 laptops," direction-free) and **RMSE** (squares errors to **penalize outliers**). Then **plot**: an **actual-vs-predicted scatter** (drop NaNs first; add a 45° line; set matching x/y limits), a **residual histogram** (is the model biased?), and an **error-vs-time line** (with a horizontal 0 line — is there a *pattern* to over/under-estimation?). Because early forecasts are warm-up, consider **chopping the start** before presenting to management. Slicing and plotting like this is far cleaner than tinkering with Excel ranges.

## 5.8 Optimization with grid search

Rather than hand-tuning, **grid-search** α (and β). Use **`np.argmin`** to find the position of the minimum error. Build empty lists for parameters, KPIs (error metrics), and outputs, then a **nested for loop** — outer over α (0.05→0.8, step 0.01), inner over β — fitting the model at each combination and storing α, β, and MAE. (75 × 75 ≈ **5,625 models** — slow but thorough; add `print` statements to *see* the nesting if it's confusing.) Then `np.argmin(MAE)` returns the position of the best model, and you report the winning α/β and its error. For the textbook example, the best was **simple** smoothing, **α = 0.4, MAE = 2.74** (or 0.41 with the finer grid). You can optimize on **MAE or RMSE** — no wrong answer, it depends on the business problem. You now have real time-series modeling chops to complement the machine learning you'll meet in later courses.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Introduction to Exponential Smoothing
:class: note dropdown
- Time-series forecasting uses the past to predict the future (demand, stocks, weather); beware **data leakage**.
- **Moving averages** weight the window equally; a longer window smooths the trend.
- **Exponential smoothing** weights recent data more, fading the past **exponentially**, via a tunable **α**.
- These are statistical **curve-fitting** models (not ML); the skill is fitting *and* **evaluating** them.
:::

:::{admonition} Motivation and initialization of Simple Exponential Smoothing
:class: note dropdown
- **Fₜ = α·Dₜ₋₁ + (1−α)·Fₜ₋₁** — blend the newest observation (learn) with the last forecast (remember).
- **α** (0–0.8): high → responsive but jumpy; low → smooth but slow; weights **drop off** exponentially.
- Reduces the impact of outliers/noise vs. a plain moving average.
- **Initialize** `F₀ = D₀` (phony but necessary to start).
:::

:::{admonition} Building the simple exponential smoothing model from scratch (and playing with alpha)
:class: note dropdown
- **Don't trust the earliest forecasts** (initialized to the actual = leakage) — focus later in the series.
- Code `simple_exp_smoothing(D, extra_periods, α)` with a **NumPy array**, `np.full(..., nan)`, and the update loop.
- Future periods **repeat the last value** (no extrapolation); return demand/forecast/error.
- α ≈ 0.4–0.5 is sharp; **α ≈ 0.99 collapses to a moving average**.
:::

:::{admonition} Wrapping up Simple Exponential Smoothing
:class: note dropdown
- Expanding the recursion shows the forecast weights **all** past observations — hence "exponential."
- As **α → 1**, it's a window-1 moving average.
- **Limitations:** flat constant future forecast — **no trend** (need double ES), **no seasonality** (triple), **no external drivers**.
- Simple isn't bad — **evaluation** is what adds value.
:::

:::{admonition} Introduction to Double Exponential Smoothing
:class: note dropdown
- Adds **trend** via **β**: **forecast = level + trend**, with **Fₜ₊λ = level + λ·trend** (extrapolates a slope).
- Level = α·Dₜ + (1−α)·(prev level + trend); trend = β·(change) + (1−β)·(prev trend).
- Initialize **level = D₀**, **trend = D₁ − D₀** (first forecast perfect, then degrades).
- Adds a **constant trend forever**, so it loses value over long horizons.
:::

:::{admonition} Double exponential smoothing from scratch
:class: note dropdown
- Keep three NaN arrays — **level `A`**, **trend `B`**, **forecast `F`**.
- Update level `A[t] = α·D[t-1] + (1-α)·(A[t-1]+B[t-1])`; update trend `B[t] = β·(A[t]-A[t-1]) + (1-β)·B[t-1]`.
- The future carries the **last trend** forward — it **breaks** over a long horizon (`extra_periods=15`).
- Good for **short** forecast horizons.
:::

:::{admonition} Evaluating how your model did
:class: note dropdown
- **error = actual − predicted** (a residual; overestimate = negative) — state your **sign convention** to the audience.
- **MAE** = average miss in problem units (direction-free); **RMSE** squares errors to penalize outliers.
- Plot **actual-vs-predicted** (drop NaNs, add a 45° line, matching limits), a **residual histogram**, and **error-vs-time**.
- Early forecasts are warm-up — consider **chopping the start** before presenting.
:::

:::{admonition} Optimization with grid search
:class: note dropdown
- Grid-search α (and β) instead of hand-tuning; **`np.argmin`** finds the minimum-error position.
- A **nested for loop** over α × β (0.05→0.8, step 0.01) fits ~5,625 models and stores each MAE.
- Report the winning α/β and error (textbook: simple ES, α=0.4, MAE=2.74).
- Optimize on **MAE or RMSE** — depends on the business problem.
:::
