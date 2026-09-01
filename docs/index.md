# Refresh / Content Opportunity Scoring: Which Pages to Review First

## Abstract

This paper asks which content pages a search team should review first each week — for
refresh, expansion, or continued monitoring. Using one month (March 2026) of the FlyRank
internship warehouse (`fact_content_daily_performance`, 331,437 pages), I built a transparent
baseline rule and a Random Forest classifier, validated with a client-holdout split to avoid
leakage across clients. The Random Forest reached a Precision@50 of **0.440** versus **0.020**
for the hand-written baseline rule (a **22x lift**) and a ROC-AUC of **0.918** on the honest,
client-grouped holdout. This is a large, directional, decision-support signal for prioritizing
reviewer attention — but as documented in Limitations below, part of that lift is likely
explained by a feature that sits close to the label's construction, not purely by the model
discovering a subtle pattern.

## Introduction / Problem Statement

Content teams can't review every page every week. The question this project answers: of all
the pages a client owns, which ones should a human reviewer look at first — and why? The
decision this supports is where to spend limited weekly review capacity. A false negative (a
real decline going unnoticed) is the more expensive failure mode, so recall on the
declining-with-demand subset matters alongside precision.

## Data

- **Source:** `FlyRank/internship-warehouse`, table `fact_content_daily_performance`
- **Window:** month = 2026-03 (a mid-panel month, not the sealed final-month test slice)
- **Pages analyzed:** 331,437 (aggregated from 9,841,378 daily rows)
- **Base rate (declining share):** 8.7% — imbalanced, notably lower than the starter CSV's
  54.2%, since this label is computed differently (see Methodology)
- **Missing data:** `avg_position` is missing for 46.7% of pages (pages that never ranked in
  a tracked position that month); filled with 0 and treated as its own signal
- **Excluded:** client identifiers used for grouping only, never as a model feature; no
  FlyRank product flags are present in this table by design; no page titles, URLs, or keyword
  text anywhere in this repo or paper

## Methodology

**Label:** `is_declining_label` — a page is labeled declining if its second-half-of-month
clicks are lower than its first-half-of-month clicks. This is a stricter, un-thresholded
definition than the FlyRank research paper's own "Trend Direction" metric (which requires a
move of more than 10% over 30-day windows). That difference is disclosed here rather than
presented as identical, and likely explains why this project's base rate (8.7%) is far lower
than the starter CSV's `trend_direction == "down"` rate (54.2%) — any decrease at all counts
here, but a page needs to clear a real threshold before FlyRank's own paper calls it "down."

**Baseline:** a transparent 0–100 score combining demand (impressions), consistency of
visibility across the month, and position opportunity — weighted 0.45 / 0.35 / 0.20, with no
machine learning involved.

**Features:** `impressions`, `clicks`, `avg_position`, `ctr`, `days_with_data` — all
whole-month aggregates. A deliberate leakage test (ML-05/ML-09) confirmed that adding
`second_half_clicks` directly as a feature pushes ROC-AUC from 0.918 to 0.996 — an inflation
of +0.078 — and that column is correctly excluded from the final feature set.

**Validation:** client-grouped holdout (11 of 55 clients held out, 61,638 of 331,437 rows). A
side-by-side check against a naive random row-level split showed the naive split reports an
optimistic ROC-AUC of 0.951 versus the honest grouped split's 0.918 (a +0.033 gap) — the
grouped number is the one reported throughout this paper.

**Model:** Random Forest classifier (class-balanced, `max_depth=10`, `min_samples_leaf=25`,
`n_estimators=200`), compared against Logistic Regression and the baseline rule on the same
holdout split and the same metric.

## Results (vs Baseline)

| Model | ROC AUC | Precision@20 | Precision@50 | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| baseline_rule | 0.818 | 0.00 | 0.020 | - | - |
| logistic_regression | 0.885 | 0.30 | 0.360 | 0.811 | 0.544 |
| random_forest | 0.918 | 0.40 | 0.440 | 0.999 | 0.619 |

Random Forest's Precision@50 (0.440) is a **22x lift** over the baseline rule (0.020) and
roughly **5x** the natural base rate (8.7%). Feature importances: `ctr` (0.404), `clicks`
(0.380), `impressions` (0.145), `avg_position` (0.058), `days_with_data` (0.014).

**A caveat on these numbers, in the spirit of the ML-09 validation audit:** `clicks` is the
whole-month click total, and the label is built by comparing `first_half_clicks` to
`second_half_clicks` — two components that sum to `clicks`. While `clicks` was not one of the
columns explicitly excluded as label-derived (only the half-month splits were), it sits close
enough to the label's construction that it likely explains a meaningful share of the model's
strength, alongside `ctr`'s near-identical importance. A stronger follow-up would drop
`clicks` entirely and re-validate on `impressions`, `avg_position`, `ctr`, and
`days_with_data` alone.

## Signal Audit — A Genuine Surprise

Before modeling, I hypothesized three directions a hand-written rule might assume: declining
pages would have worse average position, lower CTR, and less consistent visibility than
non-declining pages. **All three came back opposite the hypothesis:**

| Signal | Not-declining mean | Declining mean | Difference |
|---|---:|---:|---:|
| avg_position | 16.93 | 11.28 | **-5.64** (declining pages rank *better*) |
| ctr | 0.002 | 0.012 | **+0.010** (declining pages have *higher* CTR) |
| days_with_data | 29.57 | 30.93 | **+1.36** (declining pages are *more* consistently visible) |

This is a genuine finding, not a modeling error: on this month's slice, pages currently
declining tend to be pages that were already doing *well* — well-positioned, well-clicked,
consistently visible — and are now seeing clicks concentrate more in the first half of the
month than the second. This reframes the practical story: this label may be catching
early-stage softening on strong pages rather than "obviously struggling" pages, which is a
different (and arguably more useful) signal for a reviewer than expected going in.

## Limitations

- Single-month proxy label (current-window decline, not a validated future-window outcome),
  and stricter than the FlyRank paper's own 10%-threshold trend definition (see Methodology)
- **The `clicks` feature sits close to the label's construction** (see Results caveat above);
  reported lift numbers should be read with that in mind. A version of this model with
  `clicks` removed was tested separately and still showed a strong lift (Precision@50 rose
  to 0.540, a 27x lift over baseline), suggesting the effect isn't purely an artifact of that
  one feature, though `ctr` alone then carried the majority of the model's weight (0.687
  importance) — a signal worth continued scrutiny in any follow-up
- Modest holdout size: only 11 test clients out of 55 total
- No causal claim: this shows association with decline, not that refreshing a page causes
  recovery — that requires an experiment
- Observable signals only; no FlyRank product decision flags were used or available
- All claims here are observed / measured / directional / decision-support, never guarantees
  for any individual page

## Ranked Recommendations

The final queue blends the model's probability (70%) with the baseline rule score (30%) into
one 0–100 score, then maps each page to one of four actions, each with a reason code attached.

**Action mix across all 331,437 scored pages:**

| Action | Pages |
|---|---:|
| monitor | 162,924 |
| monitor_closely | 104,397 |
| refresh_and_review_ctr | 40,523 |
| refresh | 23,593 |

64,116 pages (19.3%) were flagged as high-priority (`refresh` or `refresh_and_review_ctr`).

**Intended use:** a weekly reviewer aid for 55 clients' worth of content, not an automated
publishing system. A human must confirm the reason code by opening the actual page before
acting. Auto-publishing, auto-pruning, or auto-de-indexing based on this score alone is
explicitly out of scope.

**Retrain trigger:** re-run monthly on the newest mid-panel month; retrain if next month's
holdout Precision@50 drops notably below this month's 0.440, or if the base rate drifts
outside roughly the -1% to 19% range around this month's 8.7%.

## Reproducibility

- Notebooks: `work/notebooks/w01_research_question.ipynb` through
  `work/notebooks/capstone.ipynb`
- Random seed: 42, used consistently across all splits and models
- Environment: `pip install -r requirements.txt`, plus `duckdb`, `huggingface_hub`,
  `pandas`, `scikit-learn`, `matplotlib` for the warehouse notebooks
- Data access: [`FlyRank/internship-warehouse`](https://huggingface.co/datasets/FlyRank/internship-warehouse)
  on Hugging Face (gated, instant approval)
- To reproduce: clone this repo, request warehouse access, run each notebook in
  `work/notebooks/` top to bottom in order

## Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset. Data and pipeline courtesy of
[FlyRank](https://flyrank.ai). Research findings referenced from FlyRank's
["The State of AI-Driven SEO" data report, April 2026](https://state-of-seo-2026.flyrank.ai/).
