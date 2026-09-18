# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Azka Asif
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** azka-asif/FlyRank-ML
- **Date:** 18 September 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. Sections 1–8
> mirror the Pass / Needs-Work rubric axes, so nothing here is optional. Sections 0 and 9
> are **paper sections**: your deployed research paper must carry both, and they're here so
> you never rebuild them from memory at ship time.

## 0. Abstract

This project looks at whether recent search performance can be used to prioritise content pages for refresh review. 
I used page-level data from the FlyRank ML Internship warehouse and created monthly observations so that recent performance could be compared with what happened in the following month. 
I compared a simple trend-based baseline with logistic regression and Random Forest using a time-based test period. 
The Random Forest performed best at the top of the ranking, with a Precision@50 of 78% compared with a 69.2% decline rate in the test set, although its ROC AUC was only 0.571. 
The final output is a ranked review queue with reason codes that can help editors decide which pages may be worth checking first.

## 1. Problem framing

The main goal of this project is to decide which content pages should be reviewed first for a possible refresh. 
Since it is not practical to manually check every page, the project uses recent search performance to rank pages by priority.
Each observation represents a content page in a particular month. The output is a ranked list with an opportunity score and reason codes showing why a page appears higher in the queue. 
The ranking is meant to help an editor decide where to look first rather than automatically deciding that a page needs to be changed.
A wrong recommendation mainly costs editorial time. A page that is not actually a strong refresh opportunity could be reviewed before another page where attention may have been more useful.

## 2. Data safety

For the final project I used the FlyRank ML Internship warehouse, mainly the daily content performance table. 
I aggregated the daily data into monthly page-level observations and used data from December 2025 to June 2026. 
After creating the history and future windows and applying the minimum-impression filter, the final modelling dataset had 291,517 observations from 43 clients and 105,697 content pages.
I only kept rows where Google Search Console data was available, and I removed pages with fewer than 100 impressions in the observation month because percentage changes can become unstable when the numbers are very small.
Client and content IDs were only used for grouping and joining and were not included as model features. 
I also excluded anything that directly contained information about the future outcome, such as `next_impressions` and the `future_decline` label. 
I did not use label-derived fields such as `trend_direction` or `trend_pct` as model features because that would leak information about the outcome. 
No client names, domains, URLs or raw search queries are included in the analysis or report.

## 3. Baseline

I first created a simple trend-based baseline so that the machine-learning models had something straightforward to compare against. 
The baseline assumes that pages already losing impressions are more likely to keep declining in the following month.
The baseline score is based on the negative value of recent impression change. 
This makes it easy to understand and gives a fair comparison because it is tested on the same May-to-June period and with the same ranking metric as the models.
On the final test period, the baseline achieved a Precision@50 of 46%. 
The overall decline rate in the test set was already 69.2%, so the baseline did not do a good job of putting the most likely declining pages at the top of the list.

## 4. Model / analysis

I treated the task as a prediction and ranking problem. For each page, the model uses recent search performance to estimate whether the page will experience a meaningful decline in the following month.
I defined the target as a future decline in impressions of at least 20% compared with the current month. 
This is only a proxy for pages that may be worth reviewing, not a direct label saying that a page definitely needs a refresh.
The features used were current impressions, clicks, CTR, average position, days with impressions, recent impression change, earlier impression change, click change, CTR change and position change.
I compared two models: Logistic Regression and Random Forest. Logistic regression was included as a simpler and more interpretable model, while Random Forest was used to capture more complex relationships between the features.
The final evaluation used a time-based split. Earlier months were used for training, and May 2026 was used to predict whether pages declined in June 2026. 
This was done so that the model only used information that would have been available before the outcome period.

## 5. Evaluation

The final evaluation used a time-based split so that the model was tested on a later period rather than on randomly mixed rows. 
The training set contained 203,633 observations, while the final test set contained 87,884 observations. 
The decline rate was 45.4% in the training data and 69.2% in the test period, so the later period was noticeably different from the earlier months.
I compared the baseline, Logistic Regression and Random Forest using the same test set. 
Since the main goal is to rank pages for review, I used Precision@50 as the main metric and also reported ROC AUC to show overall ranking quality.

| Method | Precision@50 | ROC AUC |
|---|---:|---:|
| Test-set base rate | 69.2% | — |
| Simple baseline | 46.0% | 0.493 |
| Logistic Regression | 42.0% | 0.545 |
| Random Forest | 78.0% | 0.571 |

The Random Forest performed best for Precision@50. Of the 50 pages it ranked as highest risk, 78% experienced the defined decline in the following month. 
However, the ROC AUC was only 0.571, so the model was much more useful for concentrating likely declines near the top of the list than for separating declining and non-declining pages across the full test set.

## 6. Interpretation

The Random Forest relied most heavily on `days_with_impressions`, which had a much higher feature importance than the other variables. 
CTR, position change and recent impression change were also among the more important features.
I would be careful about interpreting `days_with_impressions` as a direct search ranking signal. 
It may be capturing useful information about how consistently a page appears in search, but it could also partly reflect differences in data coverage or time periods.
Another important result was that the decline rate changed quite a lot across months. 
It was around 19.6% for February observations, about 55% for March and April, and 69.2% for May observations predicting June. 
This suggests that the prediction problem was not equally distributed across time.
The overall result is therefore fairly narrow: the Random Forest was useful for ranking a small group of pages near the top of the review list, but the low ROC AUC means it should not be treated as a strong general classifier for every page.

## 7. Recommendation

I turned the Random Forest output into a ranked review queue so that the result is easier to use in practice. 
The final opportunity score combines 70% model decline risk with 30% current search visibility. I chose these weights as a simple prioritisation rule, so they should be treated as a project design choice rather than something learned by the model.
Pages at the top of the list are marked as `review first`, followed by a smaller `review` group, while the rest are left as `monitor`. 
The ranking also includes simple reason codes such as high visibility, recent impression drop, worsening position, low CTR, or recent recovery.
The highest-ranked pages were generally pages that were already declining while still receiving meaningful search visibility. 
An editor could use the queue to decide which pages to inspect first, then manually check whether the change is related to outdated content, search intent, competition, seasonality, or another factor before making any changes.
My confidence is mainly in the ranking near the top of the list rather than in the model as a general classifier. 
The recommendations should therefore be used as decision support, not as an automatic instruction to refresh a page.

## 8. Reproducibility

The main analysis is in `work/notebooks/capstone.ipynb`.
The project was run in Python using DuckDB, pandas, NumPy, scikit-learn and matplotlib. The main packages can be installed with:
`pip install duckdb huggingface_hub pandas numpy scikit-learn matplotlib`
The FlyRank warehouse is accessed using a Hugging Face read token saved as `HF_TOKEN` in Colab Secrets. The token itself is not written into the notebook or repository.
I used a fixed random seed of 42 for the machine-learning models. 
To rerun the project, open the capstone notebook in Google Colab, enable access to the `HF_TOKEN` secret, and use `Runtime → Run all`.
The notebook rebuilds the monthly dataset, creates the features and future-decline label, trains the baseline and models, evaluates them on the final time-based test period, and then creates the ranked review queue.
No raw private client data is stored in the repository.

## 9. Acknowledgments & data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without
