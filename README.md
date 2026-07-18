# JEE/JoSAA Rank Prediction Platform

> Turns a student's JEE score into a ranked, probability-scored list of realistic college options — built on 10 years of real JoSAA admission data, not guesswork.

---

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [The Prediction Model](#the-prediction-model)
- [Data Cleaning](#data-cleaning)
- [Quota System Handling](#quota-system-handling)
- [My Role](#my-role)
- [Challenges & What I Learned](#challenges--what-i-learned)
- [Roadmap](#roadmap)
- [Status](#status)

---

## Overview

Every year, lakhs of JEE aspirants have to guess which colleges are realistically within reach based on a rank they don't fully understand the historical context of. This platform replaces the guesswork with an actual statistical model trained on a decade of real JoSAA opening/closing rank data, and returns each student a ranked, probability-scored shortlist along with a downloadable PDF report.

## Key Features

- **Weighted regression rank prediction** — predicts a student's likely rank band using 10 years of historical JoSAA data, weighted so recent years count more than older ones.
- **Admission probability scoring** — converts the predicted rank into an actual admission-probability percentage using a z-score against a fitted Normal distribution, not just a binary yes/no cutoff.
- **Quota-aware matching** — correctly separates All-India, Home-State, and Other-State eligibility, and distinguishes IIT (JEE Advanced) quota rules from NIT/IIIT (JEE Mains) quota rules, which do not follow the same allocation logic.
- **Low-data fallback** — colleges with only one or two years of history fall back to simple interpolation instead of being run through a regression that doesn't have enough data points to be statistically reliable.
- **Auto-generated PDF report** — each student gets a personalized report with rank-probability charts, generated server-side and rendered client-side.

## Tech Stack

**Backend / Modeling**
`Python` · `FastAPI` · `NumPy` (regression math) · `Pandas` (data cleaning) · `SciPy` — `scipy.stats.norm` (probability scoring) · `Matplotlib` (`pyplot`, `patches` — chart generation) · `python-dateutil` · `Pathlib` · `re` (regex-based name normalization)

**Frontend**
`React` · `TanStack Start` / `TanStack Router` · `@react-pdf/renderer` (client-side report rendering)

## The Prediction Model

The core model is a **weighted least-squares regression** run per college/branch combination across 10 years of JoSAA opening/closing rank data, using `NumPy` for the underlying linear algebra. Recent years are weighted more heavily than older ones, since cutoff trends shift year to year (new seats, popularity swings, syllabus changes).

The raw predicted rank is then converted into an **admission probability percentage** by computing a z-score against a Normal distribution fitted to that college/branch's historical rank spread (`scipy.stats.norm`) — so a student doesn't just get "you might get in," they get an actual probability number they can compare across options.

## Data Cleaning

Real JoSAA data is not clean. College names, dates, and quota category labels were inconsistent enough that a plain string-lookup was silently failing on something as small as an extra space or a stray comma in a name. The cleaning layer, built in `Pandas`, normalizes all of this before it ever reaches the modeling step — this was one of the more time-consuming parts of the build, precisely because the failures were silent rather than loud.

## Quota System Handling

JoSAA's quota system is not uniform across institute types:

- **All-India vs Home-State vs Other-State** eligibility differs by category and by the student's home state.
- **IIT seats (JEE Advanced)** and **NIT/IIIT seats (JEE Mains)** follow *different* quota rule sets entirely — a common source of prediction errors in naive models that treat all institutes the same way.

The model encodes these distinctions explicitly rather than assuming a single quota logic across every institute.

## My Role

Founder & Data Lead. I designed the regression and probability-scoring approach, built the data-cleaning pipeline, and modeled the quota-eligibility logic. The `FastAPI` backend and PDF report pipeline were built with AI-assisted tooling for implementation speed, with the underlying statistical design and edge-case handling (low-data fallback, quota logic) being my own decisions.

## Challenges & What I Learned

- Silent data-quality bugs (a name mismatched by one space) are harder to catch than loud ones — normalization needs to happen before any lookup, not as an afterthought.
- A regression model is only as trustworthy as its data volume — forcing every college through the same model regardless of history length produces confidently wrong answers for low-data colleges.
- Probability framing (a %) is far more useful to a 17-year-old making a decision than a raw predicted rank number with no context.

## Roadmap

- Extend historical window beyond 10 years where data allows
- Add confidence intervals alongside the point probability estimate
- Cover state-level engineering counselling (not just JoSAA) in a future version

## Status

Actively developed. This repository is a working build, not a finished/polished open-source release — expect ongoing changes.

---
*Built by Nusrat Ali — [LinkedIn](https://www.linkedin.com/in/nusrat-ali-47073b329)*
