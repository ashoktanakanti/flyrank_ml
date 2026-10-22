# FlyRank ML Internship: Starter Repo

Applied search intelligence: Google Search ranking and discoverability.

This repo is the shared starting point for the FlyRank ML internship. Each intern works in their own copy, does the assignments there, and submits that repo's URL. The data here is a small anonymized sample of FlyRank search data. It is enough to run the full workflow without credentials or private client data.

New here? Read [SETUP.md](SETUP.md) first (GitHub, Colab and data access). Then read [GUIDE.md](GUIDE.md) for what each file is and which ones to edit.

---

## Quickstart

Open Notebook 1 in Google Colab and run all the cells:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/notebooks/01_first_look_and_discovery.ipynb?flush_cache=true) Week 1: first look and discovery

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/notebooks/02_your_first_readable_model.ipynb?flush_cache=true) Week 2: a readable model

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/notebooks/03_working_with_the_full_release.ipynb?flush_cache=true) Weeks 3 and later: the full release, about 79 million rows, queried with DuckDB without downloading it. The release is hosted at [FlyRank/internship-warehouse](https://huggingface.co/datasets/FlyRank/internship-warehouse). It is gated: you request access and accept the data-use terms.

### Assignment notebooks

Each assignment has a notebook in `work/notebooks/`. Open its badge, fill in the sections in order, then choose **File → Save a copy in GitHub**. The dialog is pre-filled with your repo and the notebook's path.

Once you create your copy, an automatic commit ("Point Colab badges at this copy") changes the badges to open your copy instead of the shared read-only notebooks. The `personalize` workflow does this. Until that commit lands, the badges on the shared starter page open blank previews.

| Week | Card | Notebook | Open |
|---|---|---|---|
| 1 | ML-02 | `w01_research_question` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w01_research_question.ipynb?flush_cache=true) |
| 2 | ML-03 | `w02_ml_task_framing` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w02_ml_task_framing.ipynb?flush_cache=true) |
| 3 | ML-04 | `w03_data_contract` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w03_data_contract.ipynb?flush_cache=true) |
| 3 | ML-05 | `w03_feature_leakage_check` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w03_feature_leakage_check.ipynb?flush_cache=true) |
| 4 | ML-06 | `w04_signal_audit` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w04_signal_audit.ipynb?flush_cache=true) |
| 4 | ML-07 | `w04_baseline_score` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w04_baseline_score.ipynb?flush_cache=true) |
| 5 | ML-08 | `w05_model` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w05_model.ipynb?flush_cache=true) |
| 6 | ML-09 | `w06_validation_audit` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w06_validation_audit.ipynb?flush_cache=true) |
| 7 | ML-10 | `w07_action_playbook` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/w07_action_playbook.ipynb?flush_cache=true) |
| 8 | ML-11 | `capstone` | [Open](https://colab.research.google.com/github/ashoktanakanti/flyrank_ml/blob/main/work/notebooks/capstone.ipynb?flush_cache=true) |

If the badge doesn't open your copy, use Colab's own opener: **File → Open notebook → GitHub**, paste `github.com/<you>/<your-repo>`, and pick the notebook.

### Running locally

```bash
pip install -r requirements.txt
python scripts/run_all.py
```

The script runs the whole pipeline on the bundled sample and writes results to `outputs/`. This overwrites the example files that are committed there.

---

## What's in the repo

| Path | Contents |
|---|---|
| `notebooks/` | The Week 1 and Week 2 notebooks, which run in Colab |
| `scripts/` | The pipeline: `01_prepare_features.py` through `05_build_pdf_report.py`, plus `run_all.py` |
| `data/raw/content_refresh_anonymized.csv` | The anonymized starter dataset: 30,000 rows and 44 columns |
| `outputs/` | Example outputs, including `model_report.md`, a sample queue and charts |
| `work/` | Your workspace for experiments and the capstone (see `work/README.md`) |
| `docs/` | Background reading and the data dictionary |
| `skills/` | Instruction files for an AI assistant, one per task (start at `skills/README.md`) |
| `submission/` | Your capstone paper and the URL where it's deployed |

Read these in `docs/`:

1. `ml-core-foundation-framework.md`: how an ML system fits together end to end.
2. `ml-intern-dataset-and-lane-guide.md`: working with the data, the capstone workflow, and the analysis lanes.
3. `intern-free-tooling-guide.md`: the tools used in this course. They're free.
4. `data-dictionary.md`: every column in the starter dataset, with its meaning, scale and known issues.

---

## The pipeline

```text
01_prepare_features.py      cleans the data and builds features; defines the label
02_baseline_score.py        a hand-written rule that ranks pages
03_train_model.py           logistic regression, decision tree and random forest, with a client-holdout split
04_evaluate_and_export.py   the ranked queue, charts and a Markdown report
05_build_pdf_report.py      a PDF summary
```

### Results on the bundled sample

The numbers below come from the committed `outputs/model_report.md`. The sample has 30,000 rows, and 54.2% of them carry the `is_declining_label` target.

| Model | ROC AUC | Precision@50 |
|---|---:|---:|
| Hand-written rules (baseline) | 0.627 | 0.240 |
| Logistic regression | 0.700 | 0.400 |
| Decision tree | 0.742 | 0.540 |
| Random forest (selected) | 0.750 | 0.740 |

Precision@50 is the share of the top 50 ranked pages that carry the target label. The random forest has the best value on the committed run.

Re-running the pipeline with the library versions installed for this repo's requirements gave the same baseline value (0.240) but a random forest value of 0.680. The script fixes its random state, so the gap probably comes from differences in library versions. I haven't confirmed the exact cause. Quote the number from your own run, and say which versions you used.

---

## Data safety

Read [DATA_USE.md](DATA_USE.md) before you start.

- The repo contains only the anonymized starter CSV. It has no client names, domains, URLs, titles or keywords.
- Don't add raw client data to this repo or to your fork. If you need more data, ask your mentor for an approved release.
- Don't paste client data into third-party AI tools.
- Describe results as observed, measured, directional or decision support. Don't say you predicted Google's algorithm.

`.gitignore` blocks everything under `data/` except the starter CSV, and blocks parquet and archive files. The `smoke-test` workflow fails if the repo tracks any other CSV, or any archive.

---

## Workflows

- **smoke-test** runs the full pipeline on the sample and checks that the expected outputs exist.
- **personalize** rewrites the Colab badge links to point at your copy of the repo.
- **data-path-smoke** runs notebook 03 against the hosted release every Monday. It needs a token, and it's skipped in forks.

---

## Assignments and schedule

Weekly assignments, live sessions and the capstone are on your portal board. Your enrollment email has the link. This repo supplies the technical foundation for all of them.

### Setting up your submission repo

1. Create a free GitHub account.
2. Make your own public copy of this repo. Use **Use this template** and choose **Create a new repository**.
3. In Colab, open a notebook from your copy's badges and choose **File → Save a copy in GitHub**. The dialog is already pre-filled, so click OK.
4. Submit your repo URL (`github.com/<you>/<your-repo>`) for Assignment 1. Don't submit a Colab or Google Drive link.

Track leads: Mirza Ašćerić (ML) and Hole (data engineering). Code is under the MIT license (see `LICENSE`). Data is covered by `DATA_USE.md`.
