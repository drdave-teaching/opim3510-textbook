# Chapter 1 — Foundations: Python, For Loops & Distributions

Welcome to **OPIM 3510 — Business Data Analytics I (BDA1)**, your solid data course. The goal isn't just modeling — it's to make you **data-competent**: able to read data, merge files, describe data quantitatively, make beautiful plots and tables, and run statistics to say whether things are *different* or not. This chapter lays the groundwork: the **`for` loop** (the control-flow workhorse), and **distributions** (the language of applied statistics) — both of which we'll need immediately for the Central Limit Theorem in Chapter 2.

## 1.1 Welcome

Dr. Dave Wanik — professor in Operations & Information Management and academic director for Business Data Analytics — built this course from the bottom up to make you comfortable *with data*: reading it, merging it, summarizing it, visualizing it, and reasoning about it statistically. Basic modeling is here too, but competence with data is the point.

## 1.2 Distributions

A **distribution** is just a collection of numbers. You know the **normal** (bell curve), but heights, weights, salaries, and vote shares all follow distributions. Two broad kinds: **continuous** (any real value, e.g. 0.5739) and **discrete** (integers: 0, 1, 2, 3 — count data). We describe any distribution — whatever its shape — with **percentiles**: the *10th percentile* is the point at or below which 10% of values fall (e.g. "10% of people make \$50k or less"), which captures both the **center** and the **tails**.

Generate one with **`np.random.normal(loc, scale, size)`** — `loc` = mean, `scale` = standard deviation (mind the keyword names; and beware **positional-argument order** — `(3, 20, 100)` means mean 3, sd 20, size 100, a classic newbie bug). The default is the **standard normal** (mean 0, sd 1). The **empirical rule** governs the normal: ~**68%** of values within ±1 sd, ~**95%** within ±2 sd, ~**99.7%** within ±3 sd — so for mean 100, sd 20, values between 80–120 are common and anything below 40 or above 160 is very rare. Draw 10,000 samples and the histogram *is* a bell curve. (Foreshadowing: the **means** of samples drawn from *any* distribution are themselves normally distributed — the Central Limit Theorem, next chapter.)

## 1.3 For loops

We're building to the CLT, where we'll **draw samples one at a time, compute their means, and store them** — that needs a **`for` loop**. The loop iterates a set number of operations (same idea in Python, R, C, Java). Two forms:

- **Over a list:** `for i in my_list: print(i)` walks each element.
- **Over a range:** `for i in np.arange(low, high, step)` — `arange` is *low, high (excluded), step*, so `np.arange(0, 5, 1)` → 0,1,2,3,4.

`for` is a **reserved word** (shown in purple) — you can't name a variable `for`, but the loop variable can be anything (`i`, `j`, `x`, `cat`); `i`/`j` are conventional (they once meant rows/columns). A powerful pattern is **initialize a variable and reuse it in the loop** — e.g. **compound interest**: start with \$100, and each year `current = current * 1.05`; after 5 years it's \$127.63, because interest earns interest. A **nested for loop** (an inner loop completing for each outer step) lets you iterate over two things at once — useful, if a bit advanced.

## 1.4 Other common distributions

Beyond the normal (a **parametric** distribution set by mean + sd), a few more you'll use:

- **Triangular** — `np.random.triangular(left, mode, right)`: **censored** at hard left/right limits (no values beyond them), which makes it great for business "sensor"-style problems where extremes are impossible.
- **Uniform (continuous)** — `np.random.uniform(low, high)`: every value between the bounds is equally likely (a rectangle); **uniform integer** is `np.random.randint`.
- **Binomial** — `np.random.binomial(n, p, size)`: the coin-flip distribution. Flip a fair coin (p=0.5) 50 times (n=50) and you *expect* ~25 heads, but 23/26/30 happen; repeat the experiment `size` times to see the distribution. (Discrete → think count data.)

Finally, **PDF vs. CDF**. The **PDF** (what a KDE/histogram shows) is the *shape* — you can see it's normal, but the y-axis is frequency, so you can't read "P(≤ 104)". The **CDF** (cumulative) plots the **percentile** on the y-axis, so you *can*: "90% chance of drawing 102 or less." Pull exact cutoffs with the **percentile** function (min = 0th, max = 100th). Same data, two views — and the CDF is often the more useful one for an audience.

:::{admonition} 🔗 Notebooks for this chapter
:class: seealso dropdown

- **Intro to Data Science topics** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module1_EDA/1_WelcomeToClass/0_Intro%20to%20Data%20Science%20Topics.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module1_EDA/1_WelcomeToClass/0_Intro%20to%20Data%20Science%20Topics.ipynb)
- **Practical Python programming basics** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module1_EDA/1_WelcomeToClass/1_Practical_Python%20Programming%20Basics.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module1_EDA/1_WelcomeToClass/1_Practical_Python%20Programming%20Basics.ipynb)
- **Introduction to distributions & for loops** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/1_Distributions/1_Introduction%20to%20Distributions%20and%20For%20Loops.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/1_Distributions/1_Introduction%20to%20Distributions%20and%20For%20Loops.ipynb)
- **For loops** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/1_Distributions/2_ForLoops.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Module3_Stats_Geospatial/1_Distributions/2_ForLoops.ipynb)

*Exercises, assignments & extras:*

- **Assignment 1 — selecting data & statistics** &nbsp; [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/drdave-teaching/OPIM3510-notebooks/blob/master/Assignments/Spring%202024/Assignment1_SelectingData/Assignment1_SelectingData_Stats.ipynb) &nbsp; [GitHub](https://github.com/drdave-teaching/OPIM3510-notebooks/blob/master/Assignments/Spring%202024/Assignment1_SelectingData/Assignment1_SelectingData_Stats.ipynb)
:::

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Welcome to class
:class: note dropdown
- **OPIM 3510 = Business Data Analytics I (BDA1)** — the solid *data* course.
- Goal: be **data-competent** — read, merge, describe, plot, and run statistics on data.
- Some modeling, but competence with data is the focus.
- Built fresh from the ground up.
:::

:::{admonition} Welcome to Applied Stats
:class: note dropdown
- A **distribution** is a collection of numbers; **continuous** (any real value) vs. **discrete** (integers/counts).
- Describe any shape with **percentiles** (10th percentile = 10% of values at or below).
- Generate with **`np.random.normal(loc, scale, size)`** — watch positional-argument order.
- **Empirical rule**: ~68% within ±1 sd, ~95% within ±2, ~99.7% within ±3; sample means → normal (CLT, next).
:::

:::{admonition} For loops — simple and nested
:class: note dropdown
- A **`for` loop** iterates a set number of operations; `for i in my_list` or `for i in np.arange(low, high, step)`.
- `np.arange` is *low, high (excluded), step*; `for` is a **reserved word**, loop variable is free (`i`/`j`/`x`).
- **Initialize-and-reuse** pattern powers compound interest (`current *= 1.05` each year).
- **Nested loops** iterate over two things at once.
:::

:::{admonition} Other common distributions
:class: note dropdown
- **Triangular** (`left, mode, right`) is **censored** at hard limits — good for business "can't exceed" problems.
- **Uniform** (continuous or integer via `randint`) makes every value in range equally likely.
- **Binomial** (`n, p, size`) is the coin-flip distribution (50 flips, p=0.5 → ~25 heads).
- **PDF** shows the shape; **CDF** shows the **percentile** (read "P(≤ x)"); get cutoffs with the percentile function.
:::
