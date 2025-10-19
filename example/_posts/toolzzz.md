---
layout: post
title: "Text-Lab: Sentiment & Topic Modeling (R + Jupyter)"
date: 2025-10-12
description: >
  A reusable, notebook-first workflow for scoring sentiment, discovering topics, and producing one-click reports from social or web text. Built for VS Code + Jupyter, powered by tidytext.
excerpt_separator: <!--more-->
sitemap: true
tags: [tools]
categories: [tools]
---

🚀 What you get

Ready-to-run notebooks (R kernel) for:

Tidy tokenization → Sentiment (AFINN / bing / NRC) → Negation (bigrams)

TF-IDF (term importance) → Topic modeling (LDA) → Dashboards/plots

Adapter notebook to ingest any CSV/JSONL your scraper produces

Parameterized report notebook you can rerun with different inputs via papermill

Optional tiny Shiny app (upload a CSV → click to explore)

🧱 Tech stack

R 4.x (via Conda env), JupyterLab (R kernel = IRkernel)

tidyverse, tidytext, textdata, tokenizers, topicmodels, widyr, ggraph

papermill (parameterized notebooks)

(Optional) twarc2 CLI for X/Twitter API ingestion

📁 Project structure
SentimentRR/
├─ env/
│  └─ environment.yml            # conda env (R + Jupyter + core libs)
├─ data/
│  ├─ raw/                       # your scraper/API dumps (csv/jsonl)
│  └─ processed/                 # canonical csv (id,text,created_at,…)
├─ outputs/
│  ├─ intermediate/              # tokens, hashtags, etc.
│  └─ reports/                   # executed papermill notebooks
├─ notebooks/                    # step-by-step workflow
│  ├─ 00_bootstrap.ipynb         # (optional) makes folders + sample data
│  ├─ 10_ingest_sample.ipynb     # loads sample csv
│  ├─ 11_ingest_custom.ipynb     # **adapter** for your scraper
│  ├─ 20_tidy_text.ipynb         # tokens (one token per row) + hashtags
│  ├─ 30_sentiment.ipynb         # AFINN/bing/NRC (+ time series)
│  ├─ 40_tfidf.ipynb             # term importance by group
│  ├─ 50_ngrams_negation.ipynb   # "not good" → polarity flip (AFINN)
│  ├─ 60_topic_modeling.ipynb    # LDA topics (+ sentiment by topic)
│  └─ 90_exports.ipynb           # optional export utilities
├─ templates/
│  └─ 70_report_template.ipynb   # **parameterized report**
└─ app/
   └─ app.R                      # (optional) Shiny mini-app

🧪 Quick start (daily)
conda activate r-text-lab
cd "C:\Users\giannis\Desktop\Project\moneyzz\SentimentRR"
jupyter lab


Run notebooks in order:

11_ingest_custom.ipynb → produces data/processed/tweets_flat.csv (id, text, created_at).

20_tidy_text.ipynb → saves tokens to outputs/intermediate/.

30_sentiment.ipynb → per-post sentiment + time series.

40_tfidf.ipynb, 50_ngrams_negation.ipynb, 60_topic_modeling.ipynb.

Generate a reusable report (AFINN example):

papermill templates\70_report_template.ipynb `
  outputs\reports\run_afinn.ipynb `
  -p DATA_PATH data\processed\tweets_flat.csv `
  -p LEXICON afinn -p NEGATION true -p TIMEBIN day `
  -p OUTPUT_CSV outputs\sentiment_afinn.csv `
  -p TITLE "AFINN — Daily Report"

📥 Ingesting your data (your scraper ➜ notebooks)

Data contract: your file can be anything, as long as we can map to:

id (string) — unique post id

text (string) — full text

created_at (datetime, optional) — helps for time series

author_id (optional), lang (optional)

Use notebooks/11_ingest_custom.ipynb to adapt any CSV/JSONL:

Set INPUT_PATH to your file

Map your columns (e.g. tweet_id → id, full_text → text)

Run the cell → writes data/processed/tweets_flat.csv for the rest of the pipeline

🧠 What the analysis does
Tidy text

Rule: one token per row.

Removes stop-words; keeps hashtags separately for feature analysis.

Sentiment

AFINN (−5…+5 intensity), bing (±), NRC (emotions).

Per-post scores → optional hour/day aggregates if created_at exists.

Negation handling: bigrams flip AFINN when preceded by not/no/never/without.

TF-IDF

Surfaces important terms per hashtag/account/timebin (not just frequent).

Topics (LDA)

Discovers themes across posts; you can compute sentiment by topic by joining topic terms to AFINN.

Reporting

The parameterized notebook lets you switch input file, lexicon, negation, and timebin without editing code, saving an executed report + CSV.

🔧 Usage snippets (copy–paste)
Run all notebooks (manual)

Open Jupyter → run 11_ingest_custom → 20_tidy_text → 30_sentiment → 50_ngrams_negation → 60_topic_modeling.

One-click report (bing, hourly)
papermill templates\70_report_template.ipynb `
  outputs\reports\run_bing_hourly.ipynb `
  -p DATA_PATH data\processed\tweets_flat.csv `
  -p LEXICON bing -p NEGATION false -p TIMEBIN hour `
  -p OUTPUT_CSV outputs\sentiment_bing_hourly.csv `
  -p TITLE "Bing — Hourly"

Optional: launch mini-app (if app/app.R exists)
Rscript -e "shiny::runApp('app')"

🔍 Example flows
10 Twitter accounts (you build the scraper)

Your scraper outputs data/raw/my_accounts.jsonl (or .csv).

11_ingest_custom.ipynb maps your columns → writes data/processed/tweets_flat.csv.

Run the analysis notebooks.

Use papermill to generate a shareable report.

RSS / Web pages

Use your own gatherer (RSS → CSV, web → CSV).

Adapt with 11_ingest_custom.ipynb and proceed.

📦 Installing / updating

Create env from YAML (already done once):

conda env create -f env\environment.yml
conda activate r-text-lab
Rscript -e "IRkernel::installspec()"


Install lexicons once (interactive R):

library(textdata)
lexicon_afinn(); lexicon_bing(); lexicon_nrc()


Update environment later:

conda env update -f env\environment.yml --prune

🧭 Troubleshooting (Windows quick fixes)

conda not found:
Run once: & "$Env:UserProfile\miniconda3\condabin\conda.bat" init powershell → reopen PowerShell.

R kernel missing in Jupyter:
Rscript -e "IRkernel::installspec()"

Lexicon license prompt in scripts:
Open R interactively and run
lexicon_afinn(); lexicon_bing(); lexicon_nrc() once.

PowerShell treats R as history:
Use Rscript (e.g., Rscript -e "...").

No timestamps in your file:
Reports still work; time-series plots will skip.

🔒 Notes on X/Twitter data

If you ingest from the X API, follow your plan’s endpoint/limits and the Developer Agreement/Policy. Prefer API/archives over web UI scraping. (If you’re ingesting other public sources, respect robots.txt and site terms.)

🗺️ Roadmap (nice-to-haves)

Emoji sentiment toggle (ADD_EMOJI) for social text

Per-account/hashtag dashboards

Scheduled runs (Windows Task Scheduler → papermill command)

Export to CSV/Parquet + compact HTML report

[⬇️ **Download the SentimentRR toolkit (ZIP)**]({{ '/assets/downloads/SentimentRR.zip' | relative_url }})
