# 🎬 Movie Analytics Dashboard

**Director popularity, genre trends & box-office performance across 10,821 films (1960–2015).**

An interactive Power BI dashboard built on the TMDB movie dataset, exploring what drives film popularity — by director, genre, and era — backed by a Python cleaning pipeline and a star-schema data model.

<img width="778" height="442" alt="image" src="https://github.com/user-attachments/assets/c67eec92-6de2-48eb-9a37-32ed7905fcc6" />

> **Two stages, one project.** This repo pairs a **Python data-cleaning & analysis** stage with a **Power BI dashboard** stage — the pipeline transforms the raw TMDB file into a clean star schema, which the dashboard then visualises.

---

## 📊 Overview

This project takes a raw, messy movie dataset (10,866 records) and turns it into a clean, model-ready analytics layer feeding a six-visual interactive dashboard. It answers questions like:

- Which directors are the most *consistently* popular (not just one-hit wonders)?
- How has average film popularity shifted from 1960 to 2015?
- Which genres dominate the catalogue, and how has the mix changed over time?
- Is there a relationship between a film's budget and its revenue?
- What's each director's single most popular film?

---

## 🗂️ Dataset

- **Source:** TMDB (The Movie Database) movie dataset
- **Raw size:** 10,866 rows × 21 columns
- **Cleaned size:** 10,821 films
- **Time span:** 1960–2015
- **Key fields:** popularity, budget/revenue (nominal + inflation-adjusted), director, genres, vote average, runtime, release year

---

## 🧹 Data Cleaning (Python / pandas)

The raw file had several issues that would quietly break a director-focused dashboard. The cleaning pipeline (`clean_movies.py`) handles them:

| Issue | Fix |
|-------|-----|
| **Corrupted names** — double-encoded UTF-8 (e.g. `Alejandro GonzÃ¡lez IÃ±Ã¡rritu`) affecting 302 directors | Repaired with `ftfy` → `Alejandro González Iñárritu` |
| **Placeholder zeros** — 52% of budgets / 55% of revenues were `0`, meaning "not reported" | Converted to blank so they don't drag averages to zero |
| **Duplicate & orphan rows** | Dropped 1 exact duplicate + 44 rows with no director |
| **Multi-value fields** — `director` and `genres` were pipe-delimited (`Kyle Balda\|Pierre Coffin`) | Split into bridge tables so co-directed films credit every director |
| **Unreliable dates** — mixed formats, year mismatches | Standardised on the clean `release_year` integer |

Derived fields added: `profit_adj`, `roi`.

---

## 🏗️ Data Model (Star Schema)

movie_director[movie_id]  ───1  movies_clean[movie_id]  1───  movie_genre[movie_id]
(bridge)                      (fact table)                    (bridge)


- **Fact table:** `movies_clean` — one row per film
- **Bridge tables:** `movie_director`, `movie_genre` — split pipe-delimited values into rows
- **Relationships:** Many-to-one, with **bidirectional cross-filtering** so a director/genre selection propagates back to the film rows

---

## 🧮 Key DAX Measures

```dax
Avg Popularity = AVERAGE(movies_clean[popularity])

Film Count = DISTINCTCOUNT(movies_clean[movie_id])   -- distinct to avoid double-counting through bridges

Top Film =
CALCULATE(
    MAX(movies_clean[original_title]),
    TOPN(1, movies_clean, movies_clean[popularity], DESC)
)

Filter Summary =
"Showing " & [Film Count] & " films (" &
MIN(movies_clean[release_year]) & "–" & MAX(movies_clean[release_year]) & ")"
```

---

## 📈 Dashboard Features

- **KPI cards** — total films, average rating, year range
- **Popularity trend** — average popularity by release year, with a tooltip surfacing each year's top film
- **Top Directors** — Top 8 by average popularity, filtered to directors with **≥ 5 films** so one-hit blockbusters don't skew the ranking
- **Genre treemap** — film count by genre
- **Budget vs Revenue scatter** — log-scaled to spread out films ranging from thousands to billions in budget
- **Genre mix over time** — stacked area of the top 8 genres by year
- **Interactive cross-filtering** — clicking any visual filters the rest

---

## ⚠️ Limitations & Caveats

Documented honestly, because knowing your data's limits matters:

- **`popularity` is TMDB's internal engagement score**, not box office or view count. Great for ranking *within* the dataset; not a normalized cross-era metric.
- **Financials cover only ~45% of films** — budget/revenue visuals are filtered to reported figures and labelled accordingly.
- **Coverage is sparse before ~1990** — early-year trends sit on very few films, so those points are less stable.

---

## 🛠️ Tech Stack

- **Power BI Desktop** — data modelling, DAX, visualisation
- **Python** (pandas, ftfy) — data cleaning pipeline
- **Power Query** — final shaping on import

---

## 🚀 Reproduce

```bash
# 1. Clean the raw data
pip install pandas ftfy
python clean_movies.py        # outputs movies_clean.csv, movie_director.csv, movie_genre.csv

# 2. Open the .pbix in Power BI Desktop and point the data sources
#    at the three generated CSVs
```

---

## 📁 Repository Structure

├── data/
│   ├── Unfiltered_Movies.csv      # raw source
│   ├── movies_clean.csv           # cleaned fact table
│   ├── movie_director.csv         # director bridge
│   └── movie_genre.csv            # genre bridge
├── clean_movies.py                # cleaning pipeline
├── MovieAnalytics.pbix            # Power BI dashboard
├── assets/
│   └── dashboard.png              # dashboard screenshot
└── README.md

---

## 👤 Author

**Sadnan** — Computer Science, Toronto Metropolitan University
[GitHub](https://github.com/saddiboi) · [Project Repo](https://github.com/saddiboi/Movies_Data_Analysis_PowerBI)

---

*Built as a data analytics portfolio project. Dataset courtesy of TMDB.*
