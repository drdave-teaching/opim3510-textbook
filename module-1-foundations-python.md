# Chapter 1 — Foundations: Data Science & Python Basics

Welcome to data science. This chapter sets the stage: **what data science is**, the difference between **structured and unstructured** data, the **five data-science jobs** you might grow into, and enough **Python** to start coding — data types, arithmetic, booleans, strings, and lists — all inside **Google Colab**. Dr. Dave's promise: anyone can learn to code ("Python is the new Microsoft Word of our generation"), if you put in ~20 minutes a day. The course arc runs EDA → regression → classification → advanced techniques → a taste of text analytics.

## 1.1 What is data science?

Data science is a blend of three things: **computer science** (making data talk to models and deploying them), **math and statistics** (combining information, recoding, fitting models), and **domain knowledge** (the business context that makes it useful). Being a "data scientist for its own sake" is shallow — the magic is a data scientist *with a domain* (energy, finance, marketing, insurance…). There are three flavors of analytics: **descriptive** (data storytelling — plots and tables for insight), **predictive** (using history to forecast), and **prescriptive** (optimization); this course focuses on the first two. The digital economy runs on data — recommendation engines, self-driving cars, sentiment analysis — and models aren't just as accurate as people, they're **vastly faster** (reading 1,000 reviews in a second, tirelessly and consistently).

## 1.2 Structured vs. unstructured data

Most of the course works with **structured ("spreadsheet") data** — rows and columns of categorical and numeric fields. Spreadsheets run the world but get **clunky and error-prone** past a handful of rows (hidden sheets, broken macros, no audit trail); a notebook lays out your logic clearly instead. We'll slice, wrangle, tabulate, plot, and model this tabular data all semester.

**Unstructured data** — text, images, time series — is "the new oil," where the state of the art (and many jobs) is heading. The key idea: everything becomes a **numeric representation** a computer can chew on. **Text** → **one-hot encoding** (1 if a word appears, else 0), then richer schemes like **TF-IDF** (how *common* a word is) and **n-grams** (unigrams/bigrams/trigrams for context). **Images** are secretly data too — each pixel is a brightness value **0–255** — so algorithms detect edges and textures to classify who's in a photo or what a house will sell for. **Time series** (stocks, demand) increasingly use flexible ML over rigid statistical assumptions (e.g. Facebook Prophet). The recurring theme: **domain expertise matters** — tools alone can be automated away; your business knowledge is where the value is.

## 1.3 The five data-science jobs

Be a **jack of all trades, master of one**. Five specializations that overlap:
1. **Data generalist / visualization expert** — the softer, storytelling side (compelling infographics from stats).
2. **Data wrangler / "data janitor"** — reading, merging, filling missing values, standardizing, de-duplicating, enriching (every domain needs it).
3. **Data engineer** — software/IT: data **pipelines** that move, secure, and operationalize feeds, dealing with the "**V**s" of big data (volume, velocity, variety, and more).
4. **Data hunter** — the detective brokering **external** data (vendors, census, weather, government) into usable formats at good prices.
5. **Data scientist** — modeling and analysis, mostly **supervised learning** (classification and regression, where the data includes the **target** you're predicting, like house price or spam), plus **unsupervised learning** (finding natural groupings when there's no target). A **decision tree** is a friendly, transparent example — you can print the rules and read a prediction straight off it.

## 1.4 Setting up Colab

Everything lives in the **Google universe**. Use a Google/UConn email, make a class **folder** in Drive (with subfolders per module) to stay organized — nothing installs locally, so a dead laptop costs you nothing. **Google Colab** (colab.research.google.com, or add "Colaboratory" via *New → More → Connect more apps*) is a free cloud notebook that blends **text, images, video, and Python** in one place — which is why Dr. Dave teaches (and grades) in notebooks instead of PowerPoint. **Copy to Drive** to save your own version, **connect the runtime** (Google's cloud machine), and run a cell with the play button or **Ctrl+Enter**. You can **mount your Drive** to read CSVs/Excel files straight into a notebook. A good habit: **download the `.ipynb`** of each posted script for a local copy, and **read the script before lecture** to know what's coming.

## 1.5 Python as a big calculator

Python is "a big calculator you get to boss around." Import **`this`** to see the **Zen of Python** — *beautiful is better than ugly; explicit is better than implicit; simple is better than complex* — the guiding style. **Comments** start with `#` (everything to its right is ignored; render green) — as a beginner, comment nearly every line. **`print(...)`** shows output; a cell otherwise prints only its last value. Assign with a **single `=`** (`x = 3`), and note **order matters** — a cell holds the last value it ran, and re-running downstream cells inherits it. Two numeric types: **integers** (whole) and **floats** (decimals). Arithmetic: `+ - * /`, `**` for exponent, `//` for floor division. Variables can be **overwritten** (`x = x + 1`), which trips up newcomers but is just logic — the computer does exactly what you told it.

## 1.6 Booleans, strings, and lists

**Booleans** are `True`/`False` (capitalized, reserved). You rarely type them directly; instead they come from **comparisons** — and here's the crucial trio: a **single `=`** *assigns*, a **double `==`** tests "is identically equal to," and **`!=`** tests "is not equal to." So with `x=1, y=2`, `x == y` is `False` and `x != y` is `True` — the basis for row subsetting later.

**Strings** are quoted characters — start and end with the **same** quote type (mixing them, or forgetting one, errors with something like "EOL while scanning string literal"). You can `len()` them, **concatenate** with `+` (add a space yourself), substitute with `%s`, and apply grammar operations: `.upper()`, `.lower()`, `.replace(old, new)`, `.strip()` (trim whitespace) — `replace` is handy for cleaning stray `?`s or `999`s.

**Variable names**: keep them **lowercase** (Colab hides your variables, so consistency avoids confusion), must **start with a letter or underscore**, and use only letters/numbers/underscores — no leading digits, dashes, spaces, or `?`.

**Lists** are general containers (`[...]`) that hold **mixed types**. Python **counts from 0**, so `x[1]` is the *second* element; you'll practice **slicing** (ranges, "everything after," "all but the last two") — a bit idiosyncratic, and drilled later. Next chapter: pandas and real spreadsheet data.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Hi from Dave!
:class: note dropdown
- Dr. Dave teaches grad stats/optimization/ML/deep learning; this is data science for undergrads, all in **Python**.
- "Python is the new Microsoft Word of our generation" — anyone can learn it with ~20 minutes of practice a day.
- Course arc: data wrangling & visualization → **EDA** → **regression** → **classification** → advanced techniques → text analytics.
- Most of the class is **spreadsheet data**, with a text-analytics unit at the end.
:::

:::{admonition} Structured vs. unstructured data
:class: note dropdown
- The course focuses on **structured (spreadsheet)** data; Excel gets clunky and error-prone past a few rows.
- **Unstructured** data (text, images, time series) is "the new oil" — everything becomes a **numeric representation**.
- Text → **one-hot encoding** (word present = 1), then **TF-IDF** (word rarity) and **n-grams** (context).
- Images are data too — each pixel is a **0–255** brightness value; **domain expertise** keeps you from being automated away.
:::

:::{admonition} The five data science jobs
:class: note dropdown
- Be a **jack of all trades, master of one**; five overlapping roles.
- Visualization generalist, **data wrangler/janitor**, data engineer (pipelines + the "Vs"), data hunter (external data), and **data scientist** (modeling).
- Data scientists mostly do **supervised learning** — regression and classification, where the data includes the **target**.
- **Unsupervised learning** finds natural groupings with no target; a **decision tree** is a transparent, readable model.
:::

:::{admonition} What is data science?
:class: note dropdown
- Data science = **computer science + math/stats + domain knowledge**; a data scientist *with a domain*, not for its own sake.
- Three analytics types: **descriptive** (storytelling), **predictive** (forecasting), **prescriptive** (optimization) — we focus on the first two.
- Models can be as accurate as people and **far faster** (1,000 reviews in a second, consistently).
- Download the `.ipynb` for a local copy; **read the script before lecture**.
:::

:::{admonition} Practical Python: Booleans, strings, and lists
:class: note dropdown
- **Booleans** are `True`/`False`; single `=` **assigns**, double `==` tests equality, `!=` tests inequality.
- **Strings** use matching quotes; `len`, `+` concatenation, `%s` substitution, `.upper/.lower/.replace/.strip`.
- **Variable names**: lowercase, start with a letter/underscore, letters/numbers/underscores only.
- **Lists** hold mixed types; Python **counts from 0**, so `x[1]` is the second element (slicing practiced later).
:::

:::{admonition} Practical Python: numeric types and arithmetic operators
:class: note dropdown
- Python is a **big calculator**; `import this` shows the **Zen of Python** (simple > complex).
- Comment nearly everything with `#`; `print()` shows output (a cell otherwise prints only its last value).
- **Integers** vs. **floats**; operators `+ - * /`, `**` (exponent), `//` (floor division).
- **Order matters** — cells inherit the last value; variables can be **overwritten** (`x = x + 1`).
:::

:::{admonition} Setting up Colab
:class: note dropdown
- Everything lives in **Google Drive**; make a class folder with per-module subfolders — nothing installs locally.
- **Google Colab** is a free cloud notebook blending text, images, video, and Python.
- **Copy to Drive**, **connect the runtime**, run cells with the play button or **Ctrl+Enter**.
- **Mount Drive** to read CSV/Excel files straight into a notebook; a dead laptop costs you nothing.
:::
