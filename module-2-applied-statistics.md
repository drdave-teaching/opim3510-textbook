# Chapter 2 — Applied Statistics: Sampling, CLT & Confidence Intervals

This chapter is the statistical heart of the course. We can't survey all 3.5 million people in Connecticut, but a **sample** of a few hundred tells us a lot — *if* we understand the **sampling distribution of the mean**. From it comes the **Central Limit Theorem** (the result that makes inference possible), the **standard error**, and finally **confidence intervals** — turning a single sample into a defensible "we're 95% sure the answer is between X and Y."

## 2.1 The sampling distribution of the mean

A **population** is everyone/everything you care about (all UConn students, every car battery off the line); a **sample** is the few you actually measure. Using a 1,000-person height/weight dataset as a stand-in population (mean weight ≈ 161 lb), imagine you can only stop **5 people in an elevator** — one sample gives a mean of 141, the next 129, the next 149… Each is a **sample mean**, and population mean − sample mean is the **sampling error**. Now repeat that "grab 5, take the mean" **100 times** and plot the 100 sample means: they form a **normal distribution** whose center sits right on the population mean (161). That's the magic — you don't need all 1,000 people; the **distribution of sample means is normal and centered on the truth**, and it gets tighter as the sample size grows.

## 2.2 The Central Limit Theorem

Formally: **if all samples of a given size are drawn from *any* population, the sampling distribution of the sample mean is approximately normal — and the approximation improves with larger samples.** Normal parent or not. This is what lets us move from **descriptive** statistics (mean, sd) to **inferential** statistics (use a sample to say something about the population).

The spread of that sampling distribution is the **standard error of the mean**: **SE = σ / √n**. Read the formula like a BDA student — as **n grows, SE shrinks** (toward 0), so more data → a tighter, more confident estimate. Demonstration with a **uniform** (rectangular) parent, mean 200: samples of **n = 2** give a wide, uncertain bell curve; **n = 6** tightens it; **n = 30** is tight around 200 — even though the parent was a rectangle, the sample means are **normal**.

## 2.3 CLT across distributions

The same holds for **exponential**, **normal**, and **uniform** parents alike: their sampling distributions of the mean are all normal, converging on the true population mean as n rises. With a population mean of 14.98, samples of n=2 give ~13.47, n=6 ~13.98, and **n=30 ~14.99** — which is why **30 is a "magic number"** in statistics: around there, a sample gets you very close. As n → ∞, SE → 0 and the mean-of-means lands essentially on the population mean. Next, we *slice* this sampling distribution into **confidence intervals**.

## 2.4 Point and interval estimates

One sample gives a **point estimate** — e.g. survey 50 UHD-TV buyers, mean age **52.74**. But a point estimate is falsely precise (there's sampling uncertainty). Better to give an **interval**: "we're 95% sure the mean age is between 48 and 53." A **confidence interval** is a range built from sample data such that the population parameter falls inside it at a stated **confidence level**.

It rests on **z-statistics** from the standard normal (mean 0, sd 1): **95% of sample means lie within 1.96 SE of the population mean** (that's the familiar "±2 sd"), and 99% within **2.58**. The 2.5% of probability mass in each tail is the CDF you met in Chapter 1. The interval is **x̄ ± z · (σ/√n)** — point estimate plus/minus the margin. Get any z-value from `norm.ppf` (e.g. an 80% CI uses ±1.28).

## 2.5 CI when the population σ is *known*

When σ is known (well-defined manufacturing processes), use **x̄ ± z · σ/√n**. The z-values match the confidence level: 80% → ±1.28 (10% per tail), 94% → ±1.88, 95% → ±1.96, 99% → ±2.58. Example: sample **256** store managers, mean income **\$45,420**, known population σ = **\$2,050**. The point estimate is \$45,420; the 95% CI is \$45,420 ± 1.96·(2050/√256) = **± \$251** (the **margin of error**). The interval is tight *because* n is large.

## 2.6 CI when the population σ is *unknown*

In the messy real world you rarely know σ, so use the **sample** standard deviation and swap **z → t**, with **degrees of freedom = n − 1**. Because you know less, **t-values are larger than z** (95% CI: t ≈ 1.986 vs z = 1.96) — a built-in penalty that **widens** the interval. And **small samples widen it more**: at n=90 the critical t ≈ 1.66, but at n=10 (df=9) it jumps to ≈1.83 (and much more for 99%). Example: 10 tires driven 50,000 miles, mean tread **0.32"**, sample sd **0.09** → compute df=9, the t-value, the margin, and the 95% CI (≈ 0.256"–0.384"). Sensible: ask only 10 people and your interval *should* be wide.

## 2.7 CI for a proportion

Not all data is continuous — sometimes it's yes/no (who's voting for A? who likes Python?). The **proportion** p = successes / total, and its CI needs just **p**, **n**, and a **z** critical value: **p ± z · √(p(1−p)/n)**. Example: a union needs **¾ approval** to merge; of **2,000** members, **1,600** (**80%**) plan to vote yes. The 95% CI is 0.80 ± 1.96·(0.0175) = **[0.782, 0.818]**. Since the lower bound (0.782) exceeds the 0.75 threshold, you can be confident the merger passes. Same machinery as the continuous case, applied to binary data.

## 📌 Lecture key points

*Distilled takeaways from the video lectures behind this chapter — click each to expand.*


:::{admonition} Introduction to sampling distributions of the mean
:class: note dropdown
- You can't measure a whole **population**, so you take **samples**; sample mean − population mean = **sampling error**.
- Repeat "draw a sample, take its mean" many times → the **sample means form a normal distribution** centered on the population mean.
- Larger samples → a **tighter** distribution of means (more confidence).
- Holds even when the parent distribution is normal — and, as we'll see, when it isn't.
:::

:::{admonition} Sampling distribution of the mean for a uniform distribution
:class: note dropdown
- **CLT:** sample means from *any* population are approximately normal, improving with n.
- Enables **inferential** statistics (sample → population claims), beyond descriptive stats.
- **Standard error = σ/√n**; as n grows, SE → 0 (tighter estimate).
- A **uniform** (rectangular) parent still yields **normal** sample means — wide at n=2, tight by n=30.
:::

:::{admonition} Closing thoughts on the central limit theorem
:class: note dropdown
- **Exponential, normal, and uniform** parents all give **normal** sampling distributions of the mean.
- Sample means converge on the true population mean as n rises; **n ≈ 30** gets you close (the "magic number").
- As n → ∞, **SE → 0** and the mean-of-means sits on the population mean.
- Next: slice this distribution into **margins of error and confidence intervals**.
:::

:::{admonition} Intro to point and interval estimates
:class: note dropdown
- A single sample gives a **point estimate**; an **interval estimate** adds a confidence level ("95% sure it's 48–53").
- CIs rest on **z-statistics**: 95% of sample means lie within **1.96** SE of the mean (99% within 2.58).
- Interval = **x̄ ± z · (σ/√n)**; pull any z with `norm.ppf`.
- You must assume whether the population σ is **known or unknown**.
:::

:::{admonition} CI for when population std dev is known
:class: note dropdown
- Use **x̄ ± z · σ/√n** when σ is known (well-defined processes).
- z-values by confidence: 80% ±1.28, 94% ±1.88, 95% ±1.96, 99% ±2.58.
- Example: 256 managers, mean \$45,420, σ=\$2,050 → 95% CI = ± **\$251** (the **margin of error**).
- A **large n** makes the interval tight.
:::

:::{admonition} CIs for when population std dev is unknown
:class: note dropdown
- Use the **sample** sd and swap **z → t**, with **df = n − 1**.
- **t > z** (95%: ≈1.986 vs 1.96) — a penalty that **widens** the interval because you know less.
- **Small samples widen it more** (n=90 → t≈1.66; n=10 → t≈1.83).
- Example: 10 tires, mean 0.32", sd 0.09" → 95% CI ≈ 0.256"–0.384".
:::

:::{admonition} CIs for a proportion
:class: note dropdown
- For yes/no data, **p = successes/total**; CI = **p ± z · √(p(1−p)/n)**.
- Needs only **p**, **n**, and a **z** critical value (95% → 1.96).
- Example: 1,600/2,000 = 80% yes → 95% CI **[0.782, 0.818]**.
- Lower bound above the 0.75 threshold → confident the merger passes.
:::
