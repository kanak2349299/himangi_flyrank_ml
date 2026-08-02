# Capstone Report — Ranked Content Recommendations for SEO Lift

- **Author:**Himangi Gupta
- **Lane:** Ranked Content Recommendations for SEO Lift
- **Repo:**https://github.com/kanak2349299/himangi_flyrank_ml/blob/main/work/capstone_report_template.md
- **Date:**2 August 2026


## 1. Problem framing
**Decision**: Which 20 posts/pages should the content team prioritize each week to maximize SEO lift.
**Unit of analysis**: `post_id` 
**Output**: Ranked list + predicted_score + reason_code
**Human action**: Promote top 5, Schedule next 10, Review last 5 for risk
**Cost of wrong call**: Wasting promotion budget on low-impact content. Opportunity cost of missing high-lift posts.
**Why ML**: Manual ranking can't scale to 1000s of posts. Model finds patterns in author_score, content_type, R1+R2 signals that humans miss.

## 2. Data safety
**Data used**: FlyRank Warehouse dataset via HuggingFace. Date range: 2025-01-01 to 2025-06-30
**Columns used**: author_score, content_type, R1, R2, hashtag_spam_score, publish_date
**Columns excluded**: `trend_direction`, `trend_pct` - label leakage risk. `client_name`, `email` - PII.
**Leakage check**: No future engagement used. `author_id` used only for Grouped split, not as feature.
**Pseudonymous IDs**: author_id is hashed. No client-identifying info in `work/`.

## 3. Baseline
**Baseline**: LogisticRegression with 5 core features + Random 80/20 split
**Why fair**: Same data, same features, same metric ROC-AUC and Precision@20
**Baseline numbers**: ROC-AUC = 0.68, Precision@20 = 0.45

## 4. Model / analysis
**Method**: GradientBoostingClassifier. Fits lane because it captures non-linear interactions between reason codes and author signals.
**Feature list**: author_score, content_type, R1_flag, R2_flag, hashtag_spam_score, post_length
**Features left out**: raw text, trend_pct - to avoid leakage and overfitting
**Target definition**: `seo_lift_flag = 1` if post drove >15% organic traffic lift in 30 days after publish

## 5. Evaluation
**Split**: GroupKFold by `author_id`. Why: prevents same author in train and test. More honest than random split.
**Metrics**: 
| Model | ROC-AUC | Precision@20 |
| --- | --- | --- |
| Baseline | 0.68 | 0.45 |
| GBM Honest Split | 0.69 | 0.48 |
**Error analysis**: Model over-predicts for new content_type. Under-predicts for posts with R3 spam flag.

## 6. Interpretation
**Key findings**: R1+R2 reason codes and author_score are top drivers. Video + high author_score = 2.1x higher lift.
**Action**: Use Top-20 queue. Do R1+R2 posts first. Flag R3 for manual review.

## 7. Limitations
1. Model degrades if >15% new content_types appear - needs monthly retrain
2. Only valid for 30-day SEO window. Not for long-term brand
3. Depends on quality of human R1+R2 labeling

## 8. Monitoring
**Triggers**: 
- Precision@20 < 0.40 for 2 weeks → Retrain
- >15% new content_type → Feature update
**Log**: `work/outputs/monitoring_log.csv`

## 9. Reproducibility
**Outputs**: `final_results_table.csv`, `playbook_top20.csv`, `feature_importance.png`, `reproducibility_config.json`
**Run**: `python scripts/run_model.py` then `python scripts/generate_playbook.py`

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
