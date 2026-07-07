# Chapter 4 — Classification

Now we predict **categories** instead of numbers. The good news: the **same five-step recipe** (read → split → scale → fit → evaluate) carries straight over — only two things change, the **models** (logistic regression, decision tree / random forest *classifiers*) and the **metrics**. Regression metrics measured *distance* (how far off); classification metrics measure *what proportion is correct*, built from a **confusion matrix** into **precision, recall, and F1**. We cover those metrics, then **logistic regression** theory and fitting, and finish with the tree-based classifiers and **hyperparameter tuning**.

## 4.1 From regression to classification

A quick recap of the journey: statistics and wrangling (Ch. 1), EDA and plots (Ch. 2), then the ML recipe with linear/tree/forest **regressors** and distance metrics — MAE (in the problem's units, sign removed), MSE (penalizes outliers), R² (variance explained, 0–1). Classification keeps the **read/split/scale** steps identical; what's new is fitting a **logistic regression** (and tree/forest classifiers) and evaluating with a **confusion matrix** and **classification report** (precision, recall, F1) rather than distance measures.

## 4.2 The confusion matrix

Imagine a model predicting **0/1** — 0 is the **negative** class, 1 the **positive** class (no emotional meaning, just labels). Comparing **actual** vs. **predicted**, every row is one of four cases:
- **True negative (TN):** actual 0, predicted 0.
- **True positive (TP):** actual 1, predicted 1.
- **False positive (FP):** actual 0, predicted 1.
- **False negative (FN):** actual 1, predicted 0.

Tabulated, this is the **confusion matrix** (a.k.a. a 2×2 contingency table). In scikit-learn's layout, **TN is top-left, TP is bottom-right**, with FP/FN on the off-diagonal — you want most data on the **diagonal**. Compute it with **`confusion_matrix(actual, predicted)`** (actual first, always). To avoid confusion over cell positions, **`.ravel()`** unpacks it as `(tn, fp, fn, tp)`. For a meeting, **`ConfusionMatrixDisplay`** renders a labeled, color-coded version.

## 4.3 The classification report

The four cells recombine into **fractions** that describe fit:
- **Precision** = TP / (TP + FP) — of everything predicted positive, what fraction truly is. "When the model says 1, how often is it right?"
- **Recall** = TP / (TP + FN) — of the *actual* positives, how many you caught. "Of the real 1s, how many did I find?"
- **F1** = the **harmonic mean**, `2 · precision · recall / (precision + recall)` — balances the two.

All range 0–1 (higher is better). **Accuracy** (overall correct proportion) is fine on balanced data but **misleads on imbalanced** data (90% zeros → 90% accuracy by predicting all zero, with precision/recall near 0) — which is why we prefer **F1**, especially the **weighted** F1 (weighted by **support**, the per-class sample size) for comparing models. Get it all with **`print(classification_report(actual, predicted))`** — you *must* wrap it in `print`, or the output is unreadable. Report the positive class by convention, and you can rework the formulas for the negative class.

## 4.4 Logistic regression theory

A **linear regression** (`β₁x₁ + β₂x₂ + …`) predicts on **(−∞, +∞)**, which makes no sense for a 0/1 target (you can't have a 120% or −3% chance). **Logistic regression** fits an **S-shaped curve** constrained to **[0, 1]**, outputting a **probability**. Picture "hours studied vs. passed the exam": few hours cluster near 0, many hours near 1, and the S-curve threads between — a far better fit than a straight line for binary data. To turn probabilities into classes, apply a **0.5 threshold**: ≥ 0.5 → 1, else → 0. As in regression, treat it as a **black box** — ignore the statistical elegance, care about accuracy.

To reuse Boston Housing (a regression dataset) for classification, **binarize the target**: `y = (df["medv"] > median)` → 1 if a home is above the median price, else 0. A nice lesson: any regression problem can become a classification problem — if the point estimate is poor, you can at least predict "above/below median."

## 4.5 Fitting a logistic regression

Same pipeline: `train_test_split(test_size=0.2, shuffle=True, random_state=42)` (never scramble the X/test/y order; the shared seed lets you compare models fairly), convert to NumPy, **`fit_transform`** on X_train / **`transform`** on X_test. Then **`from sklearn.linear_model import LogisticRegression`**, `lr = LogisticRegression()`, `lr = lr.fit(X_train, y_train)`, and `lr.predict(...)` returns **0/1** (already thresholded at 0.5). Evaluate with the confusion matrix and classification report — *only the model line changed*, the rest is identical. On train: recall (positive) ≈ 0.87, precision ≈ 0.84, **weighted F1 ≈ 0.85**; on test, ≈ 0.85 — **within ~10–20%**, so no overfitting. The **weighted F1** (bottom-right of the report) is the number to watch.

## 4.6 Decision tree & random forest classifiers, and tuning

Just as regression had tree/forest **regressors**, classification has **`DecisionTreeClassifier`** and **`RandomForestClassifier`** — two lines each, swapping `lr` for `dtc`/`rfc` (keep variable names systematic so you can copy-paste cleanly). Always fit on **`X_train, y_train`** (never the full X/y — that's leakage). A decision tree is **visualizable**, which makes **hyperparameter tuning** intuitive:
- **`min_samples_split`** — a node needs at least this many samples to split; larger → a **simpler** tree (pure nodes and small nodes stop splitting).
- **`max_depth`** — caps how deep the tree grows; smaller → a **shallower** tree.
- **`n_estimators`** — number of trees in the forest (default 100); more trees fit slower and **don't always improve** the test score.
- Split criteria like **Gini** and **entropy** decide the "best" split (details beyond this course).

Tuning is genuine experimentation — sometimes a bit of `max_depth`/`n_estimators` tweaking bumps the test F1, sometimes it doesn't. Pick the model with the best **test** weighted F1, remembering **parsimony**: if the simple logistic regression ties the random forest, prefer the simpler model. The random forest can hit near-perfect **train** scores (overfitting) — always judge on **test**. That completes the end-to-end classification recipe: same five steps, new models, new metrics.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} A refresher on the general modeling framework for classification
:class: note dropdown
- The **same five steps** (read → split → scale → fit → evaluate) apply to classification.
- Regression metrics measure **distance** (MAE in problem units, MSE penalizes outliers, R² = variance explained).
- Classification swaps in new **models** (logistic regression, tree/forest classifiers) and new **metrics**.
- Classification metrics measure **what proportion is correct** — precision, recall, F1 from a confusion matrix.
:::

:::{admonition} Introduction to the confusion matrix
:class: note dropdown
- 0 = **negative** class, 1 = **positive** class (labels, no emotional meaning).
- Four cases: **TN** (0→0), **TP** (1→1), **FP** (0→1), **FN** (1→0).
- `confusion_matrix(actual, predicted)` — scikit-learn puts **TN top-left, TP bottom-right**; want data on the diagonal.
- **`.ravel()`** unpacks `(tn, fp, fn, tp)`; **`ConfusionMatrixDisplay`** gives a labeled plot.
:::

:::{admonition} Introduction to the classification report
:class: note dropdown
- **Precision** = TP/(TP+FP) — "when it says 1, how often right?"; **Recall** = TP/(TP+FN) — "of real 1s, how many caught?"
- **F1** = harmonic mean `2·P·R/(P+R)`; all metrics 0–1, higher better.
- **Accuracy** misleads on imbalanced data — prefer the **weighted F1** (weighted by support) for model comparison.
- Use **`print(classification_report(actual, predicted))`** — must wrap in `print`.
:::

:::{admonition} Logistic regression for classification — basic theory
:class: note dropdown
- Linear regression predicts (−∞, +∞) — wrong for 0/1; **logistic** fits an **S-curve** constrained to [0, 1] (a probability).
- Apply a **0.5 threshold** to turn probabilities into classes; treat it as a **black box**, care about accuracy.
- Any regression problem can become classification — **binarize** the target (e.g. `medv > median` → 1/0).
- The S-curve fits binary data far better than a straight line.
:::

:::{admonition} Logistic regression for classification — fitting the model
:class: note dropdown
- Same pipeline; `LogisticRegression().fit(X_train, y_train)`; `predict` returns **0/1** (thresholded at 0.5).
- **Only the model line changes** — confusion matrix + classification report are identical to before.
- Train weighted F1 ≈ 0.85, test ≈ 0.85 — within ~10–20% means no overfitting.
- Watch the **weighted F1** (bottom-right of the report).
:::

:::{admonition} Decision Tree and Random Forest Classifier and hyperparameter tuning
:class: note dropdown
- Swap in **`DecisionTreeClassifier`** / **`RandomForestClassifier`** (two lines each); always fit on **`X_train, y_train`**.
- Tune **`min_samples_split`** (larger → simpler tree), **`max_depth`** (smaller → shallower), **`n_estimators`** (more trees, slower).
- Split criteria: **Gini** / **entropy**; a decision tree is **visualizable**, making tuning intuitive.
- Judge on the **test** weighted F1; remember **parsimony** — prefer the simpler model when scores tie.
:::
