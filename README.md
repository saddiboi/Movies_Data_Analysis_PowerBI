# 🎬 Movie Analytics Dashboard

**Director popularity, genre trends & box-office performance across 10,821 films (1960–2015).**

An interactive Power BI dashboard built on the TMDB movie dataset, exploring what drives film popularity — by director, genre, and era — backed by a Python cleaning pipeline and a star-schema data model.

![Dashboard Preview](<img width="778" height="442" alt="image" src="https://github.com/user-attachments/assets/c67eec92-6de2-48eb-9a37-32ed7905fcc6" />)

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
