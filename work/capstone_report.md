# Capstone Report — Content Opportunity Scoring for Search-Driven Content

- **Author:** Hassan Elnagar
- **Lane:** Refresh / Content Opportunity Scoring
- **Date:** September 2026

## 0. Abstract

Can observable search-performance and content signals be combined into a transparent ranking that helps a content team prioritize pages for review? I built a rule-based content opportunity score using content age, CTR, impressions, and average position from the FlyRank internship warehouse. The resulting queue ranks 71,667 eligible pages and attaches reason codes, review confidence, and a human-review action. A supervised model experiment was also tested, but its target was derived from historical CTR, making clicks and impressions target-related inputs; the validation audit therefore invalidates the 1.000 Precision@20 result as a publishable model claim. The final artifact keeps the transparent baseline as the safer decision-support mechanism and turns its ranked output into an auditable action playbook.

## 1. Problem framing

**Research question:** Can a transparent scoring rule prioritize content pages for refresh or optimization review using observable search-performance and content signals?

The decision is which pages a content/SEO practitioner should review first. The output is a ranked queue with a baseline score, reason code, review-confidence signal, and suggested human action.

The intended use is decision support. A high score does not prove that refreshing a page will increase traffic, clicks, or rankings.

## 2. Data safety

The project uses the `FlyRank/internship-warehouse` release used in the later notebooks.

- `dim_content`: 519,606 rows.
- `fact_content_daily_performance`: 78,835,655 rows.
- Daily performance coverage observed in the notebook: 2025-01-27 through 2026-06-30.
- Final content-level feature table in Week 7: 292,912 pages.
- Eligibility rule: average impressions >= 20.
- Eligible pages: 71,667 (24.47%).

The final transparent baseline uses four signals: content age, impressions, CTR, and average position.

Deleted/unpublished content is excluded. Identifiers are not model predictors. The baseline does not use future outcomes or a target label.

A leakage demonstration was explicitly performed in the data-contract work. The later supervised experiment was audited because its target was derived from CTR while clicks and impressions remained predictors.

## 3. Baseline

The transparent baseline normalizes four signals:

`0.30 × age + 0.30 × low CTR + 0.30 × impressions + 0.10 × near-page-one`

Older content, lower CTR, higher impressions, and better ranking position receive higher review priority.

The Week-4 notebook reported **Precision@20 = 0.375** for the baseline experiment.

The baseline is retained because it is simple, inspectable, and does not depend on the leaked supervised target.

## 4. Model / analysis

Week 5 tested Logistic Regression and Random Forest. The binary target was defined from the top 20% of historical CTR, and CTR itself was removed from the feature set.

The model feature list was:

- `content_age_days`
- `impressions`
- `clicks`
- `position`
- `search_volume`
- `backlinks`
- `word_count`

Both models reported **Precision@20 = 1.000**.

However, clicks and impressions are components of CTR, which defines the target. The Week-6 validation audit therefore identified a strong target-related leakage risk. The 1.000 result is kept as a diagnostic observation, not as a deployable model result.

## 5. Evaluation

The initial model experiment used a stratified 80/20 random split with `random_state=42`.

The validation audit also compared random and grouped splits and obtained Precision@100 = 1.000 in both cases. This does not rescue the experiment: the central problem is the target construction itself.

Therefore, the final paper does **not** claim a model lift or generalization from the 1.000 score. The fair conclusion is that the first supervised formulation made the target too easy to reconstruct from historical performance signals.

For a future production-grade model, the target should represent a genuinely future outcome, the feature window must precede that outcome, and the evaluation must be leakage-safe and time/group aware.

## 6. Interpretation

The signal audit found:

- **CTR: confirmed useful standalone signal.** Low CTR is associated with weaker observed organic performance.
- **Content age: mixed.** Age alone did not consistently identify refresh opportunities.
- **Search volume: mixed.** Higher search volume did not reliably identify the best opportunities by itself.

This supports a multi-signal baseline rather than a single threshold.

The supervised model's permutation importance also showed clicks and impressions dominating, which is consistent with the target-related construction and is another reason not to interpret the perfect metric as meaningful predictive power.

## 7. Recommendation

The Week-7 action playbook ranks 71,667 eligible pages.

| Action | Pages |
|---|---:|
| SUPPORT_PAGE_ONE | 39,267 |
| MONITOR | 27,123 |
| HOLD_FOR_REVIEW | 3,188 |
| REFRESH_CONTENT | 1,469 |
| IMPROVE_SNIPPET | 620 |

Recommended operating sequence:

1. **Refresh content** when age/decay and weak engagement signals align with meaningful visibility.
2. **Improve the snippet** when visibility is high but CTR is weak; inspect title, description, SERP features, and intent first.
3. **Support page-one / striking-distance pages** when authority or internal-link context may matter more than rewriting.
4. **Monitor low-volume pages** rather than over-interpreting weak evidence.
5. **Hold for review** when the available signals do not support a confident action.

Operational rule:

**Score → Rank → Explain → Human Review → Decide → Measure**

No action should be executed solely because a page received a high queue rank.

## 8. Reproducibility

The work is organized under `work/notebooks/`.

Key artifacts:

- `work/notebooks/w01_research_question.ipynb`
- `work/notebooks/w02_ml_task_framing.ipynb`
- `work/notebooks/w03_data_contract.ipynb`
- `work/notebooks/w03_feature_leakage_check.ipynb`
- `work/notebooks/w04_signal_audit.ipynb`
- `work/notebooks/w04_baseline_score.ipynb`
- `work/notebooks/w05_model.ipynb`
- `work/notebooks/w06_validation_audit.ipynb`
- `work/notebooks/w07_action_playbook.ipynb`
- `work/notebooks/capstone.ipynb`

The Week-7 notebook exports the ranked queue and action summary. The supervised experiment and validation audit use random seed `42`.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset — https://flyrank.ai

---

### Claims checklist

Claims in this paper are framed as observed, measured, directional, or decision-support findings. No causal impact is claimed, and the supervised 1.000 Precision@20 result is explicitly rejected as a final model claim because of target-related leakage.
