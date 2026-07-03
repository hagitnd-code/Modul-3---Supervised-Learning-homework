
# Model Card

## 1. Overview
- Task / business question: Predict customer churn to identify at-risk customers and implement retention strategies.
- Dataset (which option) and time range: Telco Customer Churn dataset (Option B). Data time range not specified in the original dataset.
- Target definition: Binary classification: 'churn' (1 for churn, 0 for no churn).

## 2. Metric & performance
- Primary metric and WHY (business cost of FP vs FN / over- vs under-forecast): F1-score. This metric was chosen to balance precision and recall, which is crucial in an imbalanced classification task like churn prediction where both false positives (predicting churn when a customer doesn't) and false negatives (missing actual churners) have significant business implications.
- Dumb baseline score: 0.0 (F1-score for 'most_frequent' DummyClassifier).
- Best model score (on the locked test set): Logistic Regression (tuned) F1-score: 0.6088; ROC-AUC: 0.8367.
- Did it beat the baseline meaningfully? Is it worth deploying?: Yes, the tuned Logistic Regression model significantly outperformed the baseline, indicating its potential value for deployment.

## 3. What the model relies on
- Top features and whether they make business sense: SHAP values were computed for X_test_numeric to identify feature importances. The actual interpretation of which features are 'top' requires visual inspection of the SHAP summary plot. (Further analysis of the SHAP plot is required to list specific features, but generally, tenure, contract type, and monthly charges are often strong indicators in churn datasets).
- Any feature you suspect is a leak or spurious? How did you check?: Not explicitly identified in the current analysis. No obvious leaks were detected during initial feature engineering.

## 4. Limitations & failure modes
- The 5 worst errors — what is the pattern?: The top 5 worst mistakes in the `err` dataframe consisted of 3 False Positives and 2 False Negatives. These were cases where the model had high confidence in its incorrect predictions, suggesting they might be edge cases, contain missing nuances, or involve data quality issues.
- Where would this model break?: The model might struggle with new customer segments not well represented in the training data, sudden shifts in customer behavior patterns, or significant changes in the business environment (e.g., new competitors, pricing changes).
- Stability across folds (mean +/- std): Mean cross-validation accuracy: 0.8084 (±0.0060). This indicates good stability, as the standard deviation is relatively low.
- Bias-variance read (train-vs-test gap):
    The analysis of the training accuracy (`0.8073`) versus the validation accuracy (`0.7960`) for the best Logistic Regression model indicates that the model generalizes well. The accuracies are relatively close, with a small difference that does not suggest significant overfitting or underfitting. This implies a good balance between bias and variance, meaning the model is capturing the underlying patterns in the data without memorizing the training examples too closely.

## 5. Fairness / ethics
- Could any group be systematically mis-served by this model?: Not explicitly analyzed in this study. Potential areas for concern could include demographic features (e.g., gender, if correlated with churn in a biased way) or specific service types, which would require dedicated fairness analysis.

## 6. Real world
- If deployed Monday: what would you monitor? What triggers a retrain?:
    - Monitor model performance metrics (F1-score, precision, recall, ROC-AUC) on new, incoming data.
    - Monitor feature drift (changes in distribution of input features) and concept drift (changes in the relationship between features and churn).
    - Monitor the volume and patterns of churned customers vs. predicted churn.
    - Retrain triggers: Significant drop in performance metrics, substantial data drift, new business initiatives affecting customer behavior, or a predefined schedule (e.g., monthly, quarterly).
- With two more weeks / more data, what would you do next?:
    - Deep dive into the identified worst errors to understand root causes and potentially engineer new features.
    - Conduct a thorough analysis of SHAP values to explain feature influence and validate business hypotheses.
    - Explore more advanced models (e.g., LightGBM, CatBoost) and more extensive hyperparameter tuning.
    - Investigate potential for ensemble methods.
    - Gather more recent data to assess temporal stability and generalizability.

## 7. Conclusion on Model Interrogation

The tuned Logistic Regression model continues to demonstrate robust performance for predicting customer churn. The cross-validation analysis, with a mean accuracy of **0.8084** (±**0.0060**), indicates excellent stability and consistency across different data partitions. This low standard deviation reinforces confidence in the reported performance metrics.

The train-vs-test gap analysis confirms that the model generalizes well to unseen data. With a training accuracy of **0.8073** and a validation accuracy of **0.7960**, the model shows a negligible gap, suggesting a good balance between bias and variance and no significant signs of overfitting or underfitting. This implies the model is effectively capturing the underlying patterns without memorizing the training data.

Error analysis revealed specific patterns among the top 5 worst mistakes. False Positives primarily involved long-tenure customers with low-engagement profiles (e.g., no internet service, two-year contracts) that the model incorrectly predicted to churn. Conversely, False Negatives were often senior citizens with high tenure and 'sticky' services but on month-to-month contracts, where the model under-predicted churn, possibly overlooking the inherent risk of their contract type. These findings highlight areas where the model struggles with nuanced customer loyalty and engagement characteristics, warranting further investigation and potential feature engineering.

Overall, the Logistic Regression model provides a stable, well-generalizing foundation for churn prediction. Future work should focus on understanding the identified difficult cases, leveraging SHAP values to explain feature influence and refine the model's ability to differentiate between seemingly loyal customers and those with subtle churn risks.

I first tuned Logisitic regretion on the default threshold. By the end of the exercise I noticed that the ratio of false negatives was high and I had to retain the model in order to lower the ratio. The results are shown.

