# Predicting and Profiling Potential Caravan Policyholders

Machine learning project in **KNIME** to find out **which customers are likely to buy a caravan insurance policy** and **what their profile looks like**, so that an insurance company can replace mass direct-mail campaigns with targeted ones.

> Course project – *Machine Learning*, Master's Degree in Data Science, Università degli Studi di Milano-Bicocca
> Team: Beatrice Bigoni, Davide Cavallo, Luca Rossini

📄 **Full report:** [report_team_44.pdf](report_team_44.pdf)
⚙️ **KNIME workflow:** [work_team_44.knwf](work_team_44.knwf)

---

## Key results

| Task | Result |
|---|---|
| **Classification** – best model | **Cost-sensitive XGBoost** (positive class weight 15) |
| Recall on buyers (test set) | **0.66**, vs about 0.10 for models trained on resampled data |
| Accuracy (test set) | about 0.74 |
| Threshold tuning | lowering the decision threshold to 0.3 raises recall to 0.88 |
| **Clustering** – best model | **K-Medoids** on behavioural and ownership variables |
| Best customer segment | **12.1% purchase rate, twice the 6% average** |

## Research questions

1. **RQ1 – Classification:** can we accurately predict which customers will buy a caravan insurance policy?
2. **RQ2 – Profiling:** which characteristics drive the purchase, and can we find customer segments with a higher propensity to buy?

## Dataset

**CoIL Challenge 2000** data from the Dutch data mining company Sentient Machine Research (also known as the *Caravan Insurance Challenge*):

- **9,822 customers** and 87 variables, no missing values;
- 43 socio-demographic variables aggregated by postal code (`M…`), 21 policy contribution variables (`P…`) and 21 policy count variables (`A…`), mostly categorical;
- target `CARAVAN` (1 = owns a caravan policy), **heavily imbalanced: only 6% positives**.

The dataset is not included in this repository; it is publicly available as *caravan-insurance-challenge.csv* (e.g. on Kaggle) and as the *Insurance Company Benchmark (COIL 2000)* in the UCI Machine Learning Repository.

## Approach

### RQ1 – Classification

1. **Feature selection**
   - *Filter method*: correlation matrix, removal of one variable from each pair with |ρ| > 0.85 (85 → 61 predictors);
   - *Embedded method*: Random Forest, ranking features by how often they are used at the root split (top 20). The 61-feature set was kept because it performed as well.
2. **Encoding**: ordinal codes treated as categories (string conversion), since the distance between levels is not linear.
3. **Data polishing**: removal of inconsistent records (identical features, different target) before balancing.
4. **Class imbalance**, two strategies compared:
   - *Data level*: random oversampling and undersampling;
   - *Algorithm level (cost-sensitive learning)*: XGBoost with `scale_pos_weight` (1, 2, 3, 15 – the imbalance ratio) and threshold optimisation for the other models with the *Binary Classification Inspector*.
5. **Models**: Decision Tree (pruned and unpruned), Random Forest, Naive Bayes, XGBoost.
6. **Evaluation**: 75/25 train-test split, stratified 10-fold cross-validation, ROC/AUC, accuracy, Cohen's Kappa, recall.

The near-perfect scores obtained with oversampling were recognised as **data leakage** (duplicated minority records ending up in both training and validation folds), which is why the cost-sensitive XGBoost was chosen as the final model: it maximises recall, the metric that matters when missing a potential buyer costs more than contacting an uninterested one.

### RQ2 – Customer segmentation

1. Min-max normalisation and one-hot encoding.
2. **Domain-knowledge feature engineering**: the 41 customer subtypes (`MOSTYPE`) grouped into 5 macro-segments with a Rule Engine (e.g. *Affluent & High-Status*, *Families & Middle-Class*).
3. **K-Means** and **K-Medoids** compared in two scenarios (with and without the aggregated socio-demographic labels).
4. Evaluation on two criteria: **silhouette coefficient** (geometric quality) and **conversion rate** of the best cluster (business value).

Removing the aggregated labels improved the cluster structure, and K-Medoids isolated a segment where the purchase rate is double the average.

## Tech stack

KNIME Analytics Platform · XGBoost · Random Forest · Decision Tree · Naive Bayes · K-Means · K-Medoids · cross-validation · cost-sensitive learning

## How to run

1. Install [KNIME Analytics Platform](https://www.knime.com/downloads).
2. Download *caravan-insurance-challenge.csv* (see [Dataset](#dataset)).
3. In KNIME: **File → Import KNIME Workflow…** and select `work_team_44.knwf`.
4. Open the **CSV Reader** node and point it to the downloaded CSV file, then run the workflow.
