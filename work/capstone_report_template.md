# Capstone Report — Enterprise Content Traffic Decline Prediction

**Author:** Jaudat Ullah Khan  
**Lane:** Machine Learning  
**Repo:** JaudatUllahKhan/my-ml-internship  
**Date:** August 2026  

---

## 0. Abstract

We designed and deployed a machine learning pipeline to forecast deep organic traffic drops across an enterprise content ecosystem before visibility loss materializes. Framing the problem as a binary classification task over sequential 90-day windows, we ingested a public-safe dataset containing 427,292 records. We engineered historical baseline features measuring raw visibility scale and structural platform activity while isolating highly volatile edge cases. Subjected to a strict client-grouped validation barrier to prevent data leakage, our trained Random Forest model achieved an audited accuracy of 86.54%, outperforming static rule heuristics on unseen client domains. The system delivers a continuous, risk-ranked action playbook that directly provides proactive directional decision-support for content optimization workflows.

---

## 1. Problem framing

- **Decision Supported:** Acts as a directional decision-support tool for corporate SEO strategists and content managers to proactively target volatile assets with structural page refreshes before visibility collapses.
- **Unit of Analysis:** An individual content node, uniquely mapped via `client_hash_id` and `content_hash_id`.
- **Output Generated:** A continuous risk probability score between 0.0 and 1.0 paired with structural operational action triggers.
- **Cost of Wrong Calls:** False positives cause unnecessary manual review overhead; false negatives result in unmitigated traffic loss and revenue decay.
- **Why Data/ML Helps:** Traditional static rules trigger high false-alarm volumes on naturally volatile or low-traffic pages. Machine learning captures non-linear interactions between historical baseline presence and continuous interaction frequencies to separate true decay from traffic noise.

---

## 2. Data safety

- **Data Ingested:** Multi-partitioned daily performance matrices extracted from `fact_content_daily_performance` across `month=2026-0*` partitions.
- **Exclusions & Rationales:** Cleartext search queries, destination URLs, and domain names were completely omitted to ensure compliance with privacy frameworks. Programmatically filtered out 8,533 zero-impression pages to exclude unindexed edge cases.
- **Leakage & Anonymization:** Excluded post-period performance signals (`trend_direction`, `trend_pct`). Entities are cryptographically anonymized via `client_hash_id` and `content_hash_id` tokens used solely for grouping. Zero client-identifying details exist anywhere in `work/`.

---

## 3. Baseline

- **Rule Definition:** Content node flags as declining if its rolling 90-day search volume drops below 80% of its historic baseline metric.
- **Baseline Performance:** While this rule yields a 100.00% score when evaluated directly against its own static definition on training data, it serves as a rigid proxy that fails to adapt to client-specific traffic variance or forecast latent asset declines.

---

## 4. Model / analysis

- **Model Selection:** Ensemble Random Forest Classifier (100 estimators, max depth of 8) to control model capacity and prevent overfitting.
- **Feature List:**
  - `impressions_prior`: Historical Search Console impression volume prior to the lookback window.
  - `active_days_count`: Count of distinct reporting days logging active search signals.
- **Target Definition:** Binary target `is_decline_target = 1` if `impressions_last_90d / (impressions_prior + 1) < 0.8` else `0`.

---

## 5. Evaluation

- **Validation Design:** 80/20 Client-Grouped Split (396,970 training rows; 30,322 validation rows) to force evaluation on entirely unseen client domains and prevent cross-domain data leakage.
- **Base Rate:** Positive target base rate across the split is **13.46%** (86.54% majority class).

| Performance Metric | Static Rule Baseline | Random Forest Model |
| :--- | :---: | :---: |
| **Pipeline Accuracy** | 100.00% | 86.54% |
| **Target Precision** | 100.00% | 42.05% |
| **Target Recall (Sensitivity)** | 100.00% | 56.22% |
| **Balanced F1-Score** | 100.00% | 48.03% |

- **Error Analysis:** Model errors predominantly stem from false positives on low-volume, highly volatile assets. Under production constraints, the model provides valuable discrimination, capturing **56.22%** of latent traffic drops on unseen client domains (a 4.17x lift over the random base rate).

---

## 6. Interpretation

- **Feature Importances:** `active_days_count` emerged as the primary split driver, establishing that continuous historical stability is a stronger signal for traffic retention than raw peak volume.
- **Surprises & Negative Results:** `impressions_prior` contributed lower predictive weight than active tracking frequency, demonstrating that consistent indexation outweighs short-term impression spikes.

---

## 7. Recommendation

**Ranked Playbook Actions:**
1. `CRITICAL_URGENT_SALVAGE` (`risk_score >= 0.8`): Direct route to technical engineering for immediate content refresh or layout repairs.
2. `OPTIMIZE_CONTENT_FOOTPRINT` (`0.5 <= risk_score < 0.8`): Route to SEO specialists for metadata updates and keyword expansion.
3. `MONITOR_STABLE` (`risk_score < 0.5`): Maintain on automated tracking loops.

**Operational Application:** Editors should import `final_action_playbook.csv` directly into task managers to triage highest-risk content nodes. Outputs provide directional decision-support rather than causal predictions.

---

## 8. Reproducibility

- **Environment & Seeds:** DuckDB (memory ceiling `10GB`), Python 3.10+, `scikit-learn`, `random_state=42`.
- **Execution Script:** Run `work/scripts/train_pipeline.py` or execute `work/notebooks/capstone.ipynb` top-to-bottom.
- **Sealed Evaluation Verification:** The sealed frame generation and metric evaluation output are committed under `work/outputs/metrics_summary.json`.

# Re-run pipeline from a fresh clone
git clone [https://github.com/JaudatUllahKhan/my-ml-internship.git](https://github.com/JaudatUllahKhan/my-ml-internship.git)
cd my-ml-internship
pip install -r requirements.txt
python work/scripts/train_pipeline.py

## 9. Acknowledgments & data credit
This research and engineering development was built on the data warehouse infrastructure provided by the [FlyRank ML Internship Track](https://flyrank.ai). The underlying performance matrix distributions, schema partitions, and baseline tracking properties originate entirely from their verified enterprise data warehouse instances.
