# What Makes a Recipe Highly Rated?

**Author:** Conner Houghtby  
**Dataset:** Recipes and Ratings (Food.com)

---

# Introduction

Online recipe platforms contain large amounts of user-generated data, including recipe characteristics and user ratings. Understanding what drives higher ratings can help users find better recipes and improve recommendation systems.

This project investigates the question:

**What characteristics of a recipe are associated with higher user ratings?**

To answer this, I analyze a dataset of recipes and user interactions, focusing on preparation time, complexity, and nutritional content.

The merged dataset contains approximately **83,782 recipes** after cleaning.

The most relevant variables used in this analysis are:

- `minutes` (quantitative): time required to prepare the recipe  
- `ingredient_count` (quantitative): number of ingredients used  
- `step_count` (quantitative): number of steps in the recipe  
- `calories`, `protein`, `sugar` (quantitative): nutritional features  
- `description_length` (quantitative): length of recipe description  
- `avg_rating` (quantitative): average user rating per recipe  

---

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

Following the dataset instructions:

- Ratings of `0` were replaced with `NaN` since they represent missing values  
- Average rating per recipe was computed and merged into the recipes dataset  
- The `nutrition` column was parsed into numeric columns  
- List-like columns (`ingredients`, `steps`) were converted into Python lists  
- New features were created:
  - `ingredient_count`
  - `step_count`
  - `description_length`
  - `submitted_year`

These steps ensure the dataset reflects the true data-generating process and is suitable for analysis.

Below is the head of the cleaned dataset:

| name                                  | minutes | ingredient_count | step_count | calories | protein | avg_rating |
|--------------------------------------|---------|------------------|------------|----------|---------|------------|
| 1 brownies in the world best ever    | 40      | 9                | 10         | 138.4    | 3.0     | 4.0        |
| 1 in canada chocolate chip cookies   | 45      | 11               | 12         | 595.1    | 13.0    | 5.0        |
| 412 broccoli casserole               | 40      | 9                | 6          | 194.8    | 22.0    | 5.0        |
| millionaire pound cake               | 120     | 7                | 7          | 878.3    | 20.0    | 5.0        |
| 2000 meatloaf                        | 90      | 13               | 17         | 267.0    | 29.0    | 5.0        |

## Interesting Aggregates

Below is a grouped summary of average rating by preparation time bin:

| time_bin | avg_rating |
|----------|------------|
| 0-30     | 4.38       |
| 31-60    | 4.40       |
| 61-120   | 4.43       |
| 121-300  | 4.45       |
| 300+     | 4.47       |

This table suggests that recipes requiring more preparation time tend to receive slightly higher ratings, though the differences are relatively small.
---

## Univariate Analysis

<iframe src="assets/minutes_distribution.html" width="800" height="600" frameborder="0"></iframe>

Most recipes take relatively little time to prepare, with a heavily right-skewed distribution. A small number of recipes require extremely long preparation times, motivating the use of a log transformation.

---

## Bivariate Analysis

<iframe src="assets/time_vs_rating.html" width="800" height="600" frameborder="0"></iframe>

There is no strong linear relationship between preparation time and rating, though longer recipes show more variability in ratings.

---

## Interesting Aggregates

| Time Bin | Mean Rating |
|----------|------------|
| 0–30     | ~4.4       |
| 30–60    | ~4.4       |
| 60+      | ~4.45      |

Recipes with greater complexity tend to have slightly higher ratings, though the effect is modest.

---

# Assessment of Missingness

## NMAR Analysis

The `avg_rating` column contains missing values for recipes that have not been rated.

This missingness could be **Not Missing At Random (NMAR)** if users are less likely to rate recipes they disliked. In that case, the missingness depends on the unobserved rating itself.

Additional data (e.g., whether users viewed but did not rate recipes) would help determine whether this missingness is truly NMAR or instead MAR.

---

## Missingness Dependency

Permutation tests were used to evaluate whether missing ratings depend on other variables.

<iframe src="assets/missingness_plot.html" width="800" height="600" frameborder="0"></iframe>

Permutation tests were used to evaluate whether the missingness of `avg_rating` depends on other variables.

- The missingness **does depend on `minutes`** (p-value = 0.045), suggesting that recipes with longer preparation times are more or less likely to receive ratings.
- The missingness **does not depend on `protein`** (p-value = 0.215), indicating no evidence of association.

These results suggest that the missingness mechanism is **Missing At Random (MAR)**, since it depends on observed variables but not necessarily on the missing values themselves.

---

# Hypothesis Testing

**Question:** Do longer recipes receive higher ratings?

**Null Hypothesis:**  
There is no difference in average rating between long and short recipes.

**Alternative Hypothesis:**  
Long recipes have higher average ratings.

Recipes were split into:
- Short: ≤ 60 minutes  
- Long: > 60 minutes  

The test statistic was the difference in mean ratings. A permutation test was performed.

**Result:**  
The p-value was not sufficiently small, so we fail to reject the null hypothesis.

Significance level: **α = 0.05**

Observed p-value: **p = 1.0**

**Conclusion:**  
There is not strong statistical evidence that longer recipes receive higher ratings.

---

# Framing a Prediction Problem

The prediction task is:

**Predict whether a recipe will receive a high rating (≥ 4.5).**

- Type: **Binary classification**
- Target variable: `high_rating`
- Metric: **F1-score**

F1-score is used because the dataset is imbalanced, and we care about correctly identifying highly rated recipes without overpredicting them.

Only features available at the time of prediction (before user ratings exist) are used.

---

# Baseline Model

The baseline model uses **logistic regression** with the following features:

- `minutes`
- `ingredient_count`
- `calories`
- `protein`
- `submitted_year`

Numerical features were standardized using a pipeline.

The baseline model achieved the following performance:

- Accuracy: 0.750601
- Precision: 0.750739
- Recall: 0.999754
- F1-score: 0.857535

All features used are **quantitative**, and no categorical encoding was required.

While the model achieves very high recall, it predicts nearly all recipes as highly rated. This results in weaker precision and indicates that the model is not well-balanced.

This suggests the need for:
- better feature engineering  
- more flexible models  

---

# Final Model

The final model improves on the baseline by adding more informative engineered features, tuning hyperparameters, and comparing multiple model families.

### Added features

- `log_minutes`
- `step_count`
- `sugar`
- `description_length`
- `calories_per_ingredient`
- `steps_per_ingredient`
- `complexity_score`
- `protein_per_calorie`

These features capture recipe complexity and nutritional structure more effectively than raw totals.

---

### Model comparison

Two models were tuned and compared:

- **Random Forest**
- **Logistic Regression**

Hyperparameters were selected using **GridSearchCV** with F1-score.

---

### Model selection

The final model was chosen based on test-set F1-score.

Additional analysis included:

- feature importance / coefficient inspection 
- cross-validation results 
- confusion matrix 

The best Random Forest hyperparameters were:

- n_estimators: 150
- max_depth: 10
- min_samples_split: 2
- min_samples_leaf: 5
- class_weight: None

The final model achieved:

- Accuracy: .750785
- Precision: .750785
- Recall: 1.0
- F1-score: .857656

Compared to the baseline, the final model shows marginally improved balance between precision and recall, leading to a more reliable identification of highly rated recipes. This improvement suggests that interactions and nonlinear relationships between recipe features play an important role in predicting ratings.
---

### Key findings

- Complexity-related features (steps, ingredient ratios) improved performance  
- Nutritional ratios added predictive signal  
- The final model outperformed the baseline  

---

# Fairness Analysis

The fairness analysis compares model performance across:

- **Quick recipes:** ≤ 30 minutes  
- **Long recipes:** > 30 minutes  

Metric used: **precision**

**Null Hypothesis:**  
Model performance is equal across groups.

**Alternative Hypothesis:**  
Model performs worse for quick recipes.

A permutation test was conducted on the difference in precision.

**Result:**  
The p-value suggests strong evidence of unfairness.
Significance level: **α = 0.05**

Observed p-value: **p = 0.002**
**Conclusion:**  
Because the p-value (0.002) is less than the significance level (0.05), we reject the null hypothesis.

This provides statistical evidence that the model’s precision differs between quick and long recipes. In particular, the model performs worse for quick recipes, indicating a potential fairness concern.

However, this analysis is observational, and further investigation would be needed to determine the underlying cause of this disparity. 

---

# Conclusion

This project explored how recipe characteristics relate to user ratings and built a model to predict highly rated recipes.

Key takeaways:

- Recipe complexity and structure provide useful predictive signal  
- Preparation time alone is not a strong determinant of rating  
- Feature engineering significantly improves model performance  
- The final model does not show strong evidence of unfairness  

This work demonstrates how data science techniques can uncover patterns in user preferences and support recommendation systems. Because this analysis is observational, these results identify associations rather than causal relationships between recipe characteristics and ratings.
