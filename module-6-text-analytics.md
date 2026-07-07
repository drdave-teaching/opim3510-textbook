# Chapter 6 — Text Analytics & NLP

The finale: teaching a computer to work with **text**. Computers can't *read* a sentence, but we can turn words into **0s and 1s** a model can chew on. Using National Weather Service **storm** narratives to predict the **event type** (hail vs. flash flood, then a multi-class version), we walk the full text pipeline — **pre-processing** (lowercase, strip punctuation, remove stop words, stem), **EDA** (word frequencies, word clouds), **feature engineering** (bag-of-words, n-grams, TF-IDF), and **modeling** — reaching **>90% accuracy**. The same five-step recipe applies; only the feature-building is new.

## 6.1 Introduction to text analytics

The goal: convert a short **event narrative** into features that predict the **event type** (the label). Terminology shift — a single sample is a **document**, the whole collection is a **corpus**. The plan for the week: **pre-process** (remove **stop words** — common, non-predictive words like *the, he, she, would*; lowercase; strip funky characters), do **EDA** (word clouds where bigger = more frequent; frequency tables/plots), **feature-engineer** (turn words into 0/1 — "does *thunderstorm* appear?" — optionally normalized by frequency with **TF-IDF**), and **model** (decision tree, random forest, or logistic regression). Even simplified to two classes we exceed 90% accuracy.

## 6.2 EDA for text and stemming

Text EDA is mostly **word frequency**. Split the narrative column into words and **`value_counts()`** the corpus for the most common terms (reported, road, water, flooding, quarter, hail…); plot them as a line graph, or — the crowd-pleaser — a **word cloud** (`wordcloud` package) that joins all documents and sizes each word by frequency. Word clouds also help **spot stop words** to remove (e.g. dropping *flash flooding* / *hail* so the model doesn't just memorize the obvious).

Then **normalize the vocabulary**. **Tokenize** (split a sentence into a list, each word given an identifier), then **stem** — chop words down to their **root** via a lookup table, not just trimming suffixes: *thunderstorms → thunderstorm*, *produced → produce*, *flooding → flood*, *crossings → cross*. Store the result in a new **`stemmed`** column; features get built from these simpler base words so the model isn't tripped up by surface variants.

## 6.3 Key pre-processing techniques

**NLP** (natural language processing) is text analytics. Three core cleanup steps, all on the narrative column:
1. **Lowercase** — `hail`/`Hail`/`HAIL` should be one token (`df["event_narrative"].str.lower()`).
2. **Remove punctuation/funky characters** — otherwise `county.` becomes a separate feature from `county`; replace non-letters with a space via a regex.
3. **Remove stop words** — common words with no predictive power, from a curated list; done with a **list comprehension** that keeps each word only if it's *not* in the stop-word list.

The toolkit is **NLTK** (Natural Language Toolkit) for processing, later paired with scikit-learn for features. You can **append custom stop words** (e.g. remove *flood* and *hail* from the descriptions) — crucial to stop the model from "cheating" on a giveaway word; be suspicious of a perfect result, and re-apply the stop words after adding them.

## 6.4 Bag-of-words and modeling

After cleaning and stemming, **join** the tokens back into space-separated strings, then treat it as a standard ML project: **`X`** = the stemmed text, **`y`** = `event_type` (~7,634 rows). Turn text into numbers with **bag-of-words (BOW)** and **n-grams**:
- **Unigram** — one word ("does *hail* appear?").
- **Bigram** — a sliding two-word window ("flash flood").
- **Trigram** — three words.

Each becomes a **0/1** feature (**one-hot encoding**), producing a mostly-zero **sparse matrix** (don't try to visualize it — check the shape and trust the process). **`CountVectorizer`** (from NLTK/scikit-learn) does the tokenizing, lowercasing, stop-word removal, and character stripping in one step, with an **`ngram_range`** to choose unigrams (~6,000 features), bigrams (~38,000), or all (~86,000). Prefer **more rows than columns** — unigrams here are the sweet spot (bigrams+ risk overfitting). Split aggressively (**60% train / 20% validation / 20% test**), fit, and predict: a decision tree hits **~98%** on both test and validation, and a **random forest** edges it out (declare it the best model).

## 6.5 TF-IDF (optional)

Instead of just presence/count, **TF-IDF** weights a word by how often it appears in a document *relative to* the whole corpus. **TF** (term frequency) = term count ÷ document length; **IDF** (inverse document frequency) = log(total documents ÷ documents containing the term); **TF-IDF** = their product, which **down-weights common words and up-weights rare** ones (rarity can be predictive). It's one line — **`TfidfVectorizer`** + `fit_transform(X)` — and here nudges accuracy to **~99%**, a small bump over bag-of-words. Worth trying both.

## 6.6 A multi-class project, and closing thoughts

To scale from two classes to **six storm types**, just **uncomment** the full dataset (`final_df`), rename it `df`, and **run all** — the entire recipe (lowercase → strip → stop words → stem → CountVectorizer/TF-IDF → 60/20/20 split → models) re-runs unchanged, because *analytics is following a recipe*. With ~33,000 rows and ~15,000 unigram features, the decision tree lands in the high 80s and the random forest higher (accuracy is imperfect here since classes are imbalanced). The **confusion matrix** is revealing: **flash flood ↔ flood** get mixed up (understandably), **hail ↔ thunderstorm** (they co-occur), **thunderstorm ↔ high wind** — but the majority sits on the diagonal. **TF-IDF** gives a small lift (89% → 92%).

That closes the course. You can now read, wrangle, and visualize data; run a structured EDA; fit and evaluate **regression** and **classification** models; interpret them with metrics and **permutation importance**; and engineer features from **text**. Text analytics is a fast-moving frontier — embeddings, summarization, and deep learning lie beyond — but you've built the foundation.

---

*This is the final chapter of the course companion. Thanks for taking the journey from `print("hello world")` to end-to-end machine learning and NLP.*

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Introduction to text analytics and project statement
:class: note dropdown
- Convert text to **0/1** features a computer can model; predict storm **event type** from the **event narrative**.
- Terminology: a sample is a **document**, the whole collection is a **corpus**.
- Pipeline: remove **stop words**, lowercase, strip characters → EDA (word clouds/frequencies) → features → model.
- Even simplified to two classes, accuracy exceeds **90%**.
:::

:::{admonition} EDA for text data and stemming
:class: note dropdown
- Text EDA is **word frequency**: `value_counts()`, frequency plots, and **word clouds** (bigger = more frequent).
- Word clouds help **spot stop words** to remove so the model doesn't memorize the obvious.
- **Tokenize** (split into a list of identified words), then **stem** to root words via a lookup table (*flooding → flood*).
- Store results in a **`stemmed`** column; build features from the simpler base words.
:::

:::{admonition} Key pre-processing techniques
:class: note dropdown
- **NLP** = text analytics; three cleanup steps: **lowercase**, **strip punctuation/funky characters**, **remove stop words**.
- Punctuation matters — `county.` would become a separate feature from `county`.
- Stop words (common, non-predictive) are removed via a **list comprehension**; the toolkit is **NLTK**.
- **Append custom stop words** (e.g. drop *flood*/*hail*) so the model doesn't cheat on giveaway words.
:::

:::{admonition} Feature engineering — bag of words — and models!
:class: note dropdown
- Join stemmed tokens back to strings; **`X`** = text, **`y`** = `event_type`.
- **Bag-of-words** with **unigrams/bigrams/trigrams** → **one-hot** 0/1 features in a **sparse matrix** (check shape, don't visualize).
- **`CountVectorizer`** does tokenizing/lowercasing/stop-words/stripping in one step, with **`ngram_range`**; prefer **more rows than columns** (unigrams).
- **60/20/20** split; decision tree ~98%, random forest edges it out.
:::

:::{admonition} (optional) TF-IDF for feature engineering
:class: note dropdown
- **TF-IDF** = term frequency × inverse document frequency — down-weights common words, up-weights **rare** (predictive) ones.
- **TF** = term count / document length; **IDF** = log(total docs / docs with term); use the **product**, not each alone.
- One line — **`TfidfVectorizer`** + `fit_transform(X)`.
- Here it nudges accuracy to ~99% — a small bump over bag-of-words; worth trying both.
:::

:::{admonition} Trying a multi-class text analytics project and closing thoughts
:class: note dropdown
- Scale to **six storm types** by uncommenting the full dataset and re-running — the recipe is unchanged.
- ~33,000 rows × ~15,000 unigram features; random forest beats the decision tree (accuracy imperfect on imbalanced classes).
- The **confusion matrix** shows sensible mix-ups (flash flood ↔ flood, hail ↔ thunderstorm); most data on the diagonal.
- **TF-IDF** gives a small lift (89% → 92%); text analytics is a fast-moving frontier beyond this course.
:::
