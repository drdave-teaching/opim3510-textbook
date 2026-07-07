# Chapter 3 — The Machine-Learning Recipe: Regression

This chapter is the heart of the course: the **machine-learning recipe**. Every supervised project follows the same **five steps** — **read → split → scale → fit → evaluate** — and once you know them, swapping in a fancier model is just one more line of code. We fit a **linear regression** on Boston Housing as a warm-up, then **decision tree** and **random forest** regressors, learning why we **split** data (the bias-variance trade-off), why we **scale** it (and how to avoid **data leakage**), and how the two tree-based algorithms actually work.

## 3.1 The general modeling framework

The five steps:
1. **Read** the data (import modules, mount Drive, check shape and missing values).
2. **Split** into a **target `y`** (the thing you predict — always `y`) and **features `X`** (everything *useful* — numeric/binary/categorical columns, **not** timestamps or random IDs), then split *those* into **train** and **test** partitions with scikit-learn's `train_test_split` (80/20 is common). You train on the train set and validate on the **unseen** test set — because models easily *memorize*, and we want patterns that **generalize**.
3. **Scale** the features to [0, 1] with `MinMaxScaler` so no large-range column dominates (and, for advanced models like neural nets, so the math doesn't diverge). Scale **X only** — never the target `y`.
4. **Fit** the model — one line: `model.fit(X_train, y_train)`.
5. **Evaluate** — the real skill. Quantitative **error metrics** (R², MAE, MSE) plus **actual-vs-predicted scatter plots** (actual always on x, predicted on y). Aim for **similar train and test performance**; a great train score with a poor test score means overfitting.

## 3.2 Linear regression on Boston Housing

A **linear regression** is an elegant *statistical* model (p-values, coefficients, inference), but in ML we treat it like a **hammer / black box** — ignore the p-values and coefficients, force it into shape, and care only about **accuracy**. It's the right *first* model: simple and fast, so you learn whether the problem is easy before reaching for something heavy.

After the recipe's first four steps (read the 506×14 data, `df.info()` for missing values, split X/y by dropping `medv` with `axis=1`, `train_test_split(test_size=0.2, random_state=...)`, convert to NumPy arrays, `MinMaxScaler`), fit with **`from sklearn.linear_model import LinearRegression`**, `lr = LinearRegression()`, `lr = lr.fit(X_train, y_train)`. Predict with `lr.predict(X_train)` (404 rows) and `lr.predict(X_test)` (102 rows). Evaluate: **R²** near 1 is best (train ≈ 0.75, test a bit lower — normal, since test is unseen); **MAE** (mean absolute error, in the problem's units — "off by ~$3,000", and it removes the +/- sign so errors don't cancel); **MSE** (penalizes outliers via squaring). Pass metrics as **`(actual, predicted)`** — never backwards. The scatter plots reveal the story: the model **underestimates the priciest homes**, but train and test look similar → a stable, useful model.

## 3.3 Decision tree and random forest regressors

Swapping models is trivial — same five steps, just new imports. Add **`DecisionTreeRegressor`** and **`RandomForestRegressor`**; each is two lines (`dtr = DecisionTreeRegressor(); dtr = dtr.fit(X_train, y_train)`), then store predictions. The **decision tree** can hit **R² = 1.0 on train** — a red flag for **overfitting** (memorizing rows); on the **test** partition it drops back toward the linear regression. The **random forest** (an ensemble of many trees) generalizes best: **R² ≈ 0.88** and **MAE ≈ $2,000** on test — the winner. The lesson: **judge models on the test partition**, be suspicious of "perfect" (0 error / R² = 1) scores, and machine-learning models beat the simple statistical one here. Compare with side-by-side train/test scatter subplots.

## 3.4 Why split, and why scale (leakage)

**Scaling** eases the math behind fitting (giant numbers make error functions explode; [0, 1] keeps everything well-behaved and gives each feature a fair shake). But there's a trap: **data leakage**. If you scale the *whole* `X` **before** splitting, you leak information about the data's distribution — the model gets to "cheat." The correct order is **split first**, then **`fit_transform` on `X_train`** (learn the coordinate system) and **`transform` on `X_test`** (apply it). Never `fit_transform` the test set. Because the scaler learned min/max from train, **test values may fall slightly outside [0, 1]** (e.g. 1.1 or −0.05) — that's fine and expected. From scikit-learn you need just three things: **`train_test_split`**, **`MinMaxScaler`** (0–1, transparent — preferred), and optionally `StandardScaler` (standard deviations from the mean). Use **`random_state`** for reproducibility and **`shuffle`** so there's no systematic order in the split.

## 3.5 MinMaxScaler mechanics

Concretely: `scaler = MinMaxScaler()`, then `X_train = scaler.fit_transform(X_train)` (overwrites with 0–1 values) and `X_test = scaler.transform(X_test)`. `describe()` confirms X_train's mins are 0 and maxes are 1 (and reveals shape — a roughly-normal column shows quartiles near 25/50/75). Scaling turns the DataFrame into a **NumPy array**, losing column names — fine, since ML cares about the *data*, not names. The **wrong way** (`fit_transform` on the full `X`, then split) leaks the distribution and produces train mins that aren't 0 — Dr. Dave will "take off all the points" for it. Fit the model on the scaled `X_train` and the **original** `y_train`.

## 3.6 The bias-variance trade-off

*Why* split at all? The **bias-variance trade-off**. A model is a black box relating features `X` to target `y`; unlike a rigid statistical form, ML models find **flexible, nonlinear** patterns. As you add **complexity**, training error keeps falling (eventually memorizing the data), but **test error** starts *rising* once the model stops generalizing. **Total error** is minimized at a sweet spot in the middle: not **underfitting** (high bias — predicting near the mean) and not **overfitting** (high variance — memorizing rows). Splitting into train/test lets us find that Goldilocks balance and confirm the model generalizes via **consistent error metrics** across partitions.

## 3.7 The decision tree algorithm

Tree-based models are where everyone starts. A **decision tree** is a flow chart of **if-then** rules. Starting at the **root node** with all the data, it greedily picks the column and split that best separates the data into groups with the lowest error (e.g. "crime < 5% → avg price $300k, else $120k"), then keeps splitting down **branches** into **leaves/nodes** until a stopping rule (no data left, or a minimum observations per split). You read a prediction straight down the tree ("Plinko"). **Pros:** very fast (a single, greedy pass) and highly **interpretable** — you can read and explain the rules. **Con:** the single pass can corner the data into partitions that miss broader nonlinear patterns — a good algorithm, not a great one.

## 3.8 The random forest algorithm

The **random forest** fixes the tree's weakness by building **many** trees (100s–1000s), each from a **random subset of rows and columns**. Each tree learns patterns on its slice; every observation "Plinkos" down all the trees, and the final prediction is the **average** of their outputs. That averaging makes it **robust and hard to break** — usually the #1 or #2 model in practice and competitions, with sensible scikit-learn defaults (tunable hyperparameters like the number of features/rows per tree). **Cons:** slower to fit than one tree, and it struggles with **high-cardinality categorical** variables. In short: split because models memorize; scale so fitting is stable; use a **decision tree regressor** for speed/interpretability and a **random forest regressor** for robust accuracy.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Intro to the General ML Modeling Framework
:class: note dropdown
- Five steps: **read → split → scale → fit → evaluate**.
- Split into target **`y`** and useful features **`X`**, then **train/test** — train to fit, test (unseen) to check **generalization**.
- **Scale X only** to [0, 1] with `MinMaxScaler`; never scale the target.
- Fit is one line; **evaluation** is the skill — R²/MAE/MSE plus actual-vs-predicted scatter; aim for similar train/test scores.
:::

:::{admonition} Fitting a linear regression on Boston Housing (Pt. 1)
:class: note dropdown
- In ML, treat linear regression as a **black box** — ignore p-values/coefficients, care only about accuracy.
- Read (506×14), check missing with `df.info()`, split X/y by dropping `medv` (`axis=1`), then `train_test_split(test_size=0.2)`.
- Convert to NumPy arrays; **`fit_transform`** on X_train, **`transform`** on X_test (never fit_transform the test set).
- Metrics: **R²** (→1 best), **MAE** (in problem units), **MSE** (penalizes outliers).
:::

:::{admonition} Fitting a linear regression on Boston Housing (Pt. 2)
:class: note dropdown
- Fit: `lr = LinearRegression(); lr = lr.fit(X_train, y_train)` — one line, runs in a blink.
- Predict train (404) and test (102); pass metrics as **`(actual, predicted)`**, never reversed.
- Train R² ≈ 0.75, MAE ≈ $3k; test slightly worse is normal — consistency means no overfitting.
- Scatter plots (actual x, predicted y) show underestimation of the priciest homes but a stable, useful model.
:::

:::{admonition} Fitting ML regression models on Boston Housing
:class: note dropdown
- Same five steps; add **`DecisionTreeRegressor`** and **`RandomForestRegressor`** (two lines each).
- A decision tree can hit **R² = 1.0 on train** — a red flag for **overfitting**; judge on the **test** partition.
- **Random forest** wins: test R² ≈ 0.88, MAE ≈ $2k; ML models beat the linear regression here.
- Be suspicious of "perfect" (0 error) scores; compare with side-by-side train/test scatter subplots.
:::

:::{admonition} Theory on scaling and partitioning your data
:class: note dropdown
- Scaling to [0, 1] keeps the fitting math stable and gives each feature a fair shake.
- **Data leakage**: scaling before splitting leaks the distribution — **split first**, then scale.
- **`fit_transform`** on train, **`transform`** on test; test values may fall slightly outside [0, 1] (fine).
- From scikit-learn: **`train_test_split`**, **`MinMaxScaler`** (preferred), `StandardScaler`; use `random_state` + `shuffle`.
:::

:::{admonition} MinMaxScaler(), fit_transform(X_train) and transform(X_test)
:class: note dropdown
- `scaler = MinMaxScaler()`; `X_train = scaler.fit_transform(X_train)`, `X_test = scaler.transform(X_test)`.
- `describe()` confirms X_train mins = 0, maxes = 1; scaling returns a **NumPy array** (names dropped — fine for ML).
- Test min/max may exceed [0, 1] because the scaler learned from train only.
- The **wrong way** (`fit_transform` on full X, then split) leaks the distribution; fit the model on scaled X_train + original y_train.
:::

:::{admonition} ML Fundamentals — bias-variance trade-off
:class: note dropdown
- ML models find **flexible, nonlinear** patterns, unlike a rigid statistical form.
- As complexity rises, **train error falls** but **test error eventually rises** — **total error** is minimized in the middle.
- **Underfitting** = high bias (predicts near the mean); **overfitting** = high variance (memorizes rows).
- Splitting into train/test finds the Goldilocks balance and confirms generalization via consistent metrics.
:::

:::{admonition} Tree-Based Models — decision tree algorithm
:class: note dropdown
- A **decision tree** is a flow chart of **if-then** rules from **root** → **branches** → **leaves**.
- It greedily picks the column/split that best separates the data (lowest error) until a stopping rule.
- Read a prediction straight down the tree ("Plinko").
- **Pros:** fast (single greedy pass) and **interpretable**; **con:** the single pass can miss broader patterns.
:::

:::{admonition} Tree-Based Models — random forest algorithm
:class: note dropdown
- A **random forest** builds many trees, each from a **random subset of rows and columns**.
- The final prediction is the **average** across all trees — robust and hard to break.
- Usually the **#1 or #2** model in practice, with good scikit-learn defaults (tunable hyperparameters).
- **Cons:** slower to fit and weak on **high-cardinality categorical** variables.
:::
