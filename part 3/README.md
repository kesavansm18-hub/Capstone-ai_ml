"# Capstone Project - Part 3" 

# Advanced Modeling & Portfolio Profitability Classification

# 1. Decision Tree Baseline Exploration

# Unconstrained vs. Controlled Structural Constraints

To establish a baseline, an unconstrained Decision Tree Classifier (unlimited depth) was contrasted against a regularization-controlled tree to mitigate high-variance over-fitting. 
   * Unconstrained Model: max_depth=None, random_state=42
   * Controlled Model: max_depth=5, min_samples_split=20, random_state=42

# Performance & Train-Test Gap Analysis

Metric               Unconstrained Tree       Controlled Tree
Training Accuracy     100.00%                     94.25%
Test Accuracy         83.00%                      80.00%
Train-Test Gap        17.00%                      14.25%

Interpretation: The unconstrained tree achieves a perfect $100\%$ training accuracy, signaling severe overfitting by memorizing specific data point paths. Implementing explicit constraints (max_depth=5) reduces the generalization gap from $17.00\%$ down to $14.25\%$, stabilizing performance across unseen cohorts.

# 2. Structural Split Criteria Math

The split performance was evaluated using two core mathematical criteria: Gini Impurity and Information Gain (Entropy).

# Mathematical Formulations
  
  * Gini Impurity: Measures the probability of a random sample being incorrectly classified if labeled according to the distribution of targets in the subset.
  
   $$Gini(p) = 1 - \sum_{i=1}^{C} p_i^2$$

  * Entropy (Information Gain): Quantifies the operational uncertainty or disorder within the target split space.
  
   $$Entropy(p) = - \sum_{i=1}^{C} p_i \log_2(p_i)$$

(Where $p_i$ represents the probability of a sample belonging to class $i$ for classes $C$.)

# Empirical Comparison
  * Gini Criterion Test Accuracy: 80.00%
  * Entropy Criterion Test Accuracy: 79.00%
The Gini Impurity criterion out-performed Information Gain by $1.00\%$ in raw test accuracy and executes faster computationally because it bypasses logarithmic calculations.

# 3. Ensemble Model Performance & Feature Ablation
# Random Forest Feature Importances
A Random Forest Classifier (n_estimators=100, max_depth=10) was trained over the feature matrix consisting of quantity (Feature 0), unit_price (Feature 1), and discount_pct (Feature 2).
  * Feature 1 (unit_price): 0.735308 (Strongest Predictor)
  * Feature 0 (quantity): 0.158053
  * Feature 2 (discount_pct): 0.106640

# Feature Ablation Experiment
  * Full Model Test ROC-AUC (All Features): 0.9346
  * Reduced Model Test ROC-AUC (1 Feature Remaining): 0.8121
  * Absolute AUC Degradation: -0.1224

# Production Trade-Off Interpretation
Dropping these features causes a major $12.24\%$ degradation in model performance. This indicates that while unit_price holds the vast majority of predictive weight, the interaction patterns between quantity and discount_pct provide critical non-linear context. From an engineering standpoint, maintaining a 3-feature ingestion matrix is highly lightweight; therefore, removing features for production optimization is unnecessary and discouraged.

# 4. 5-Fold Stratified Cross-Validation
To ensure model stability independent of train-test split fluctuations, a unified Stratified 5-Fold Cross-Validation strategy was run across all 4 candidate profiles using ROC-AUC as the core scoring metric.

--- 5-Fold Stratified Cross-Validation Performance (ROC-AUC) ---
Logistic Regression            -> Mean AUC: 0.9475 | Std Dev: ±0.0102
Decision Tree (Max Depth: 5)   -> Mean AUC: 0.9523 | Std Dev: ±0.0101
Random Forest (100 Trees)      -> Mean AUC: 0.9705 | Std Dev: ±0.0081
Gradient Boosting Machine      -> Mean AUC: 0.9767 | Std Dev: ±0.0097

# 5. Hyperparameter Tuning with GridSearchCV
Automated optimization was conducted via a robust scikit-learn pipeline wrapping data imputation, scaling, and a Random Forest Classifier.

# Hyperparameter Space Search Grid
 * randomforestclassifier__n_estimators: [50, 100, 200]
 * randomforestclassifier__max_depth: [5, 10, None]
 * randomforestclassifier__min_samples_leaf: [1, 5]

# Hyperparameter Optimization Results
 * Best Parameters Found: {'max_depth': 10, 'min_samples_leaf': 1, 'n_estimators': 100}
 * Best Cross-Validated ROC-AUC Score: 0.9695

# 6. Manual Learning Curve Analysis
The optimal tuned pipeline was sequentially exposed to progressive data availability fractions to evaluate capacity constraints versus sample size limitations.

Training Fraction	Training AUC	Test AUC
20%	                   1.0000	     0.9380
40%	                   1.0000	     0.9213
60%	                   1.0000	     0.9177
80%	                   1.0000	     0.9233
100%                   1.0000        0.9346

# Capacity vs. Data Limitations Interpretation:

The training ROC-AUC stays locked at a perfect $1.0000$ across all training sizes, while the Test AUC oscillates tightly between $0.917$ and $0.938$. This profile indicates a capacity-limited model (high variance, over-parameterized relative to the simple 3-feature dataset) rather than a data-limited one. Supplying additional data points will not fundamentally improve generalization; instead, stronger regularization or a simpler ensemble architecture is recommended to lower training memorization.

# 7. Model Serialization & Production Inference
The optimal pipeline discovered via GridSearchCV has been serialized to disk as a production asset.

 * Serialized Binary Filename: best_model.pkl
 * Verification Script Execution: A test script successfully reloaded the artifact and piped mock customer profiles containing np.nan missing flags through the pipeline. The embedded SimpleImputer fired successfully without runtime errors:

    # python Code
    # Verification Output
     --- Pipeline Inference Verification ---
     Predicted Classes     : [0 0]
     Positive Probabilities: [0.01 0.01]

# 8. Final Summary Performance Matrix

The complete performance profile across all validation horizons is synthesized below:

Model Profile	     5-Fold CV Mean AUC	 5-Fold CV Std Dev	Holdout Test-Set AUC
Logistic Regression	       0.9475	         ±0.0102	          0.9141
Controlled Decision 
Tree (depth=5)	           0.9582	         ±0.0089	          0.9173
Random Forest 
(Tuned Pipeline)	       0.9695	         ±0.0089	          0.9346
Gradient Boosting 
Machine (GBM)	           0.9767	         ±0.0097	          0.9310

