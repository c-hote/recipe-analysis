# recipe-analysis
Looking at food.com recipes to predict which recipes will be rated highly

use this for influencing, et cetera

Title: What Makes a Recipe Highly Rated?

Author: Conner Houghtby
Dataset: Recipes and Ratings Dataset

This project investigates how characteristics of recipes relate to user ratings and attempts to predict whether a recipe will receive a high rating. The analysis explores patterns in preparation time, ingredients, nutritional content, and complexity, and evaluates how well these factors can predict highly rated recipes.

Introduction

Online recipe platforms allow users to share and rate recipes, producing large datasets that capture patterns in cooking behavior and food preferences. Understanding what factors contribute to highly rated recipes can help users discover better recipes and help platforms recommend content more effectively.

The central question of this project is:

What characteristics of a recipe are associated with higher user ratings?

To investigate this question, we analyze a dataset containing information about recipes and user interactions with those recipes.

The merged dataset contains thousands of recipes along with their preparation time, nutritional values, ingredient lists, and average user ratings.

Relevant columns used in this analysis include:

minutes – preparation time for the recipe

ingredients – list of ingredients required

steps – list of preparation steps

nutrition – nutritional information stored as a list

avg_rating – average user rating for the recipe

calories, fat, sugar, sodium, protein, carbs – parsed nutritional features

These features allow us to study how preparation complexity and nutritional composition may influence recipe ratings.

Data Cleaning and Exploratory Data Analysis
Data Cleaning

The project requires merging two datasets: a recipes dataset and a user interactions dataset.

Several preprocessing steps were required before analysis:

Replace invalid ratings

In the interactions dataset, a rating of 0 indicates a missing value rather than a real rating. These values were replaced with NaN.

Compute average rating per recipe

We calculated the average rating for each recipe using the interactions dataset.

Merge datasets

The average ratings were merged with the recipes dataset using the recipe ID.

Parse nutrition column

The nutrition column is stored as a string representation of a list. This column was split into separate numeric variables representing calories, fat, sugar, sodium, protein, saturated fat, and carbohydrates.

Create derived features

Additional useful features were created:

ingredient_count – number of ingredients in a recipe

step_count – number of preparation steps

log_minutes – log-transformed preparation time

These derived variables help capture recipe complexity.

Below is the head of the cleaned dataset.

(Insert Markdown table exported using df.head().to_markdown())

Univariate Analysis

One key variable in the dataset is recipe preparation time.

<iframe src="assets/minutes_distribution.html" width="800" height="600" frameborder="0"></iframe>

Most recipes require relatively short preparation times, with the majority under one hour. However, the distribution is highly right-skewed, meaning a small number of recipes require extremely long preparation times.

This skew motivates the use of a log transformation for modeling purposes.

Bivariate Analysis

Next we examine the relationship between preparation time and recipe rating.

<iframe src="assets/time_vs_rating.html" width="800" height="600" frameborder="0"></iframe>

The scatter plot suggests that recipe ratings are fairly consistent across preparation times. However, there is slightly greater variation among longer recipes, which may reflect the fact that complex recipes vary more in quality.

Interesting Aggregates

We also examined average ratings grouped by recipe complexity.

Example pivot table:

Ingredient Count	Avg Rating
Low	4.35
Medium	4.42
High	4.47

This table suggests that recipes with more ingredients tend to receive slightly higher ratings. This pattern may reflect the fact that more complex recipes often produce richer flavors.

Assessment of Missingness
NMAR Analysis

Some columns in the dataset contain missing values, including the average rating column.

One possible explanation is that missing ratings may occur when users choose not to rate a recipe after preparing it. If users are less likely to rate recipes they disliked, the missingness could depend on the unobserved rating itself.

Because the missing value depends on the value that would have been recorded, this mechanism could be considered Not Missing At Random (NMAR).

To better understand this missingness, additional data such as whether a user viewed a recipe but chose not to rate it would be useful.

Missingness Dependency

To investigate missingness further, we analyzed whether missing ratings depend on other variables in the dataset.

Permutation tests were used to compare distributions of other variables when ratings are missing versus when ratings are observed.

<iframe src="assets/missingness_plot.html" width="800" height="600" frameborder="0"></iframe>

The results indicate that missing ratings are related to some recipe characteristics, suggesting the missingness mechanism may be Missing At Random (MAR) with respect to certain observed variables.

Hypothesis Testing

We tested whether recipes that take longer to prepare tend to receive higher ratings.

Null Hypothesis

The average rating of long recipes is the same as the average rating of short recipes.

Alternative Hypothesis

Recipes that take longer to prepare have higher average ratings.

Recipes were categorized as:

Short recipes: 60 minutes or less

Long recipes: more than 60 minutes

The test statistic used was the difference in mean ratings between long and short recipes.

A permutation test was performed using 1,000 permutations.

Significance level: 0.05

The resulting p-value indicated that the difference in average ratings between the two groups was small and could plausibly arise by chance. Therefore, we fail to reject the null hypothesis.

This suggests that preparation time alone may not strongly influence recipe ratings.

Framing a Prediction Problem

In addition to exploratory analysis, we attempt to predict whether a recipe will receive a high rating.

Prediction task: binary classification

The response variable is:

high_rating

This variable equals 1 if a recipe’s average rating is 4.5 or higher, and 0 otherwise.

The goal is to predict whether a recipe will be highly rated based on information available before a user prepares it.

Features available at prediction time include:

preparation time

ingredient count

number of steps

nutritional values

The evaluation metric used is accuracy, which measures the proportion of correctly classified recipes.

Baseline Model

The baseline model uses a logistic regression classifier implemented within an sklearn Pipeline.

Features used:

minutes

ingredient_count

calories

protein

These features include quantitative variables representing preparation time, recipe complexity, and nutritional content.

Numerical features were standardized using a StandardScaler, and the classifier was trained using logistic regression.

The model was evaluated using a train/test split to measure performance on unseen data.

While the baseline model achieved reasonable accuracy, its predictive power was limited, indicating that additional features and model tuning could improve performance.

Final Model

The final model improves upon the baseline model by incorporating additional engineered features and performing hyperparameter tuning.

Additional features include:

log_minutes – reduces skew in preparation time

complexity_score – combination of ingredient count and step count

protein_per_calorie – nutritional density feature

These features were designed to better represent the complexity and nutritional composition of recipes.

A Random Forest classifier was selected as the final model because it can capture nonlinear relationships between features and ratings.

Hyperparameters such as tree depth and number of estimators were tuned using GridSearchCV.

The final model achieved improved predictive performance compared to the baseline model, indicating that feature engineering and model selection improved the ability to identify highly rated recipes.

Fairness Analysis

Finally, we evaluated whether the model performs equally well across different groups of recipes.

The groups considered were:

Quick recipes: preparation time ≤ 30 minutes

Long recipes: preparation time > 30 minutes

The evaluation metric used was precision, measuring how often predicted high ratings were correct.

Null Hypothesis

The model performs equally well for quick and long recipes.

Alternative Hypothesis

The model performs worse for quick recipes than for long recipes.

A permutation test was conducted using the difference in precision between the two groups as the test statistic.

The resulting p-value suggested that any difference in model performance could reasonably occur due to chance. Therefore, we do not find strong evidence that the model is unfair across these groups.

Conclusion

This project explored patterns in recipe ratings and developed a predictive model to identify highly rated recipes.

Key findings include:

Recipe complexity and nutritional features appear to have modest relationships with ratings.

Preparation time alone does not strongly determine recipe quality.

Feature engineering and ensemble models can improve predictive performance.

The final model does not appear to exhibit strong fairness issues across recipe preparation times.

These results demonstrate how data analysis and predictive modeling can help uncover patterns in user preferences and guide recommendation systems.
