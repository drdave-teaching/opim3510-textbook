# OPIM 3510 — Business Data Analytics I: Data Storytelling, Applied Stats & Geospatial (book)

The hands-on companion to **OPIM 3510** — Dr. Dave Wanik, University of Connecticut.
Built from the lecture videos and Google Colab (Python) notebooks. **Work in progress:**
lecture narratives are filled in as transcripts are pulled and polished.

Live: https://drdave-teaching.github.io/opim3510-textbook/

Six chapters, each with full narrative prose and a per-lecture **📌 key points** section:

1. **Foundations & Python** — what data science is, and Python basics
2. **pandas, Data Wrangling & EDA** — DataFrames, statistics, plots, a live CA-housing EDA
3. **The Machine-Learning Recipe: Regression** — read → split → scale → fit → evaluate; trees & forests
4. **Classification** — logistic regression, the confusion matrix, precision/recall/F1
5. **Advanced Wrangling & Model Interpretation** — joins, imputation, permutation importance
6. **Text Analytics & NLP** — bag-of-words, TF-IDF, and text classification

## Build locally

```bash
pip install "jupyter-book>=0.15,<2"
jupyter-book build .
# open _build/html/index.html
```

Pushes to `main` auto-build and deploy to GitHub Pages via the `deploy-book` workflow.
