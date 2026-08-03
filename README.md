# ROGII – Wellbore Geology Prediction

Kaggle competition solution - Michaud Carlos Kouétsa
Competition organized by ROGII, deadline August 5, 2026

## 📌 Overview

This repository contains my solution to the **ROGII - Wellbore Geology Prediction** Kaggle competition, focused on automating geological interpretation along horizontal oil & gas wells.

In horizontal drilling, only the first portion of the well (right after the vertical-to-horizontal transition) has known geological position (**TVT — True Vertical Thickness**). The rest of the trajectory — often 75% of the well — must be predicted from indirect sensor measurements, mainly **Gamma Ray (GR)** signal, compared against a nearby reference well (**typewell**).

## 🎯 Task

Predict the missing TVT values for each point along a horizontal well, using:
- The well's own Gamma Ray signal and trajectory (MD, X, Y, Z)
- Signal similarity matching against a vertical reference "typewell"
- The last known TVT position as an anchor for local extrapolation

Submissions are scored using **RMSE** (Root Mean Squared Error) on a hidden test set, split into public and private leaderboards.

## 🔑 Key Findings

- Missing TVT values are **always grouped in a single block at the end of the well trajectory** (confirmed on 20 randomly sampled wells, 100% consistency) — this shifted the modeling strategy from interpolation to **extrapolation from the last known point**.
- Naive long-range extrapolation of the known trend **degrades predictions significantly** due to drift on long distances.
- Capping the extrapolation distance to a **short window (~100 ft)** performs better than longer windows — counter to initial intuition.
- Adding local trajectory variation features (direction change, dip rate) improved stability across wells.

## ⚙️ Approach

1. **Signal matching**: sliding-window normalized cross-correlation between the horizontal well's GR signal and the typewell's GR signal to estimate a "matched TVT" and confidence distance.
2. **Feature engineering**: trajectory features (MD, X, Y, Z), rolling GR statistics, distance-capped extrapolation from the last known anchor, local slope, and dip rate.
3. **Model**: LightGBM regression, validated with **GroupKFold cross-validation by well** (to avoid leaking rows from the same well between train/validation).
4. **Final model**: retrained on 100% of the training wells using the average best iteration count from cross-validation folds.

## 📊 Results

| Metric | Value |
|---|---|
| Mean CV RMSE (5-fold GroupKFold) | reported in notebook |
| Public leaderboard score | **17.013** |

## 📂 Repository Contents

- `rogii-wellbore-geology-prediction-1.ipynb` : full end-to-end pipeline (data loading → feature engineering → training → submission), executed successfully on Kaggle's infrastructure with no internet access, as required by the competition rules.
- `ROGII_note_explicative.pdf` : explanatory note (in French) covering the competition context, technical glossary, code requirements, and work log.

## 🧩 Pipeline Structure

1. Load train/test wells from Kaggle input paths
2. Build typewell sliding-window GR signatures and match against horizontal well GR signal
3. Engineer features (matched TVT, extrapolated TVT, rolling stats, trajectory variation)
4. Train LightGBM with 5-fold GroupKFold cross-validation (grouped by well to prevent leakage)
5. Retrain final model on full training data
6. Predict on test wells (only rows with missing `TVT_input`)
7. Validate submission format against `sample_submission.csv`
8. Save `submission.csv`

## 🚀 Next Steps

- Finer hyperparameter tuning
- Additional geological/spatial features
- Model ensembling to improve robustness before the August 5, 2026 deadline

## 🙏 Acknowledgements

Competition hosted by **ROGII** on Kaggle.
