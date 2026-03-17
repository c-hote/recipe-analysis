# What Makes a Recipe Highly Rated?

**Author:** Conner Houghtby  
**Dataset:** Recipes and Ratings (Food.com)

---

# Introduction

Online recipe platforms contain large amounts of user-generated data, including recipe characteristics and user ratings. Understanding what drives higher ratings can help users find better recipes and improve recommendation systems.

This project investigates the question:

**What characteristics of a recipe are associated with higher user ratings?**

To answer this, I analyze a dataset of recipes and user interactions, focusing on preparation time, complexity, and nutritional content.

The dataset contains thousands of recipes and includes variables such as:

- `minutes` (preparation time)  
- `ingredients` and `steps`  
- `nutrition` (calories, fat, sugar, etc.)  
- `avg_rating` (computed from user ratings)  

---

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

Following the dataset instructions :contentReference[oaicite:0]{index=0}:

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

The results suggest that missingness depends on observable variables such as recipe characteristics, indicating a **Missing At Random (MAR)** mechanism rather than completely random missingness.

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

The baseline model achieved reasonable performance but showed a key limitation: it predicted nearly all recipes as highly rated, leading to very high recall but less balanced performance overall.

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
The p-value suggests no strong evidence of unfairness.

**Conclusion:**  
The model does not appear to systematically disadvantage one group over the other.

---

# Conclusion

This project explored how recipe characteristics relate to user ratings and built a model to predict highly rated recipes.

Key takeaways:

- Recipe complexity and structure provide useful predictive signal  
- Preparation time alone is not a strong determinant of rating  
- Feature engineering significantly improves model performance  
- The final model does not show strong evidence of unfairness  

This work demonstrates how data science techniques can uncover patterns in user preferences and support recommendation systems.
