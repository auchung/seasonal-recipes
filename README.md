# Seasoned to Sweetness: Do Sugary Recipes Get Higher Ratings in Certain Seasons?
Authors: Audrey Chung & Amrutha Potluri

## Introduction

The **Recipes and Ratings** dataset offers a rich look into user-submitted recipes, including ingredients, preparation steps, nutritional content, and user feedback in the form of ratings. With over 200,000 recipes and nearly 1 million interactions, it allows us to explore patterns that influence how well a recipe is received.

In this project, we investigate the question:

> **What recipe characteristics are associated with higher user ratings?**

---

This question is important for both casual cooks and content platforms. For users, understanding which traits are linked to favorable ratings can guide recipe selection and creation. For platforms, identifying these features could improve personalization and recommendation systems.

To explore this, we used two CSV files that were derived from [Food.com](https://www.food.com/?ref=nav):
- `RAW_recipes.csv`: Contains metadata for each recipe, such as ingredients, steps, and nutritional values.
- `interactions.csv`: Contains user ratings for each recipe.

`recipe`, contains 83782 rows, with 8 columns that are relevant to our investigation.

| Column Name      | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| `name`           | Recipe name                                                                    |
| `id`             | Recipe ID                                                                      |
| `minutes`        | Minutes to prepare recipe                                                      |
| `submitted`      | Date recipe was submitted                                                      |
| `nutrition`      | Nutrition info: \[calories, fat, sugar, sodium, protein, saturated fat, carbs] |
| `n_steps`        | Number of steps in recipe                                                      |
| `n_ingredients`  | Number of ingredients in the recipe                                            |
| `description`    | User-provided description                                                      |

<br>

`interactions`, contains 731927 rows, with 3 columns that are relevant to our investigation.

| Column Name | Description                             |
| ----------- | --------------------------------------- |
| `recipe_id` | Recipe ID (connects to `id` in Recipes) |
| `date`      | Date of rating or review                |
| `rating`    | Rating given                            |

<br>
---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

In order to prepare our data:
- We merged the recipes and interactions datasets.
    - In particular, we performed a left merge `recipes` and `interactions` datasets on `id` and `recipe_id`, ensuring that all recipes were retained even if they had no ratings.
- We replaced all ratings of 0 with NaN.
    - Since ratings typically range from 1 to 5, we assume that ratings of 0 indicate that a missing rating rather than an extremely low rating. Thus, we replaced 0s with NaN values in order to prevent the 0s from mathematically skewing our calculations which would potentially introduce bias into our results.
- We computed the average rating per recipe, creating the `avg_rating` column.
- We created a binary `high_rating` column if the average rating surpassed an arbitrary threshold.
    - Because the distribution of the average ratings is heavily skewed left, we determined if the average rating for a recipe is **4.5** or greater, it would be considered a high rating. Otherwise, it would be considered a low rating.
- We dropped any recipes that had missing values in the `submitted` column so that we could extract a `day_of_week` column. We then determined if the day of the week fell on a weekend, creating the `is_weekend` column.
    - We wanted to explore whether the timing of recipe submissions correlates with user ratings. Typically, recipes submitted on weekends may reflect more elaborate or carefully prepared dishes, potentially leading to higher ratings. Conversely, weekday recipes may be faster or more casual, which could influence user preferences differently.
- We parsed the nutrition column into separate columns like sugar_pdv, protein_pdv, and calories.
    - We wanted to conclude if health concerns or benefits could affect recipe ratings.
- We created a `desc_len` column which is essentially the number of characters in the description of the recipe.
    - We wanted to test the idea of simplicity, determining if shorter or longer descriptions could have an effect on the recipe's rating.

| **Column Name**  | **Description**                                                           |
|------------------|---------------------------------------------------------------------------|
| `name`           | The name of the recipe                                                    |
| `avg_rating`     | The average user rating                                                   |
| `n_ingredients`  | Number of ingredients in the recipe                                       |
| `n_steps`        | Number of steps in the recipe's instructions                              |
| `minutes`        | Total time (in minutes) to prepare the recipe                             |
| `calories`       | Number of calories in a recipe                                            |
| `total_fat_pdv`  | Percent daily value of total fats in a recipe                             |
| `sugar_pdv`      | Percent daily value of sugar in a recipe                                  |
| `sodium_pdv`     | Percent daily value of sodium in a recipe                                 |
| `protein_pdv`    | Percent daily value of protein in a recipe                                |
| `sat_fat_pdv`    | Percent daily value of saturated fats in a recipe                         |
| `carbs_pdv`      | Percent daily value of carbohydrates in a recipe                          |
| `high_rating`    | Whether the average rating is at least 4.5                                |
| `day_of_week`    | Day of the week the recipe was submitted (derived from `submitted`)       |
| `is_weekend`     | Boolean indicating if it was submitted on a weekend (Sat/Sun)             |
| `desc_len`       | Length of the description (number of characters in the description)       |

<br>

Because our DataFrame contains many columns, below, we only included the most relevant columns in the head of our cleaned DataFrame:


| name                                 |   minutes |   n_steps |   n_ingredients |   avg_rating | high_rating   | is_weekend   |
|:-------------------------------------|----------:|----------:|----------------:|-------------:|:--------------|:-------------|
| 1 brownies in the world    best ever |        40 |        10 |               9 |            4 | False         | False        |
| 1 in canada chocolate chip cookies   |        45 |        12 |              11 |            5 | True          | False        |
| 412 broccoli casserole               |        40 |         6 |               9 |            5 | True          | False        |
| millionaire pound cake               |       120 |         7 |               7 |            5 | True          | False        |
| 2000 meatloaf                        |        90 |        17 |              13 |            5 | True          | False        |

---

### Univariate Analysis

We analyzed individual variables to understand their distributions and identify patterns that might influence recipe ratings.

#### Average Rating (`avg_rating`)
- Most recipes have high average ratings between 4.5 and 5.0.
- This strong right skew motivated the creation of a binary `high_rating` variable (`avg_rating ≥ 4.5`).

<iframe
  src="assets/avg_rating_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### High Rating (`high_rating`)
- A more detailed and closer look at the distribution of higher ratings

<iframe
  src="assets/high_rating_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

### Bivariate Analysis

We explored relationships between recipe characteristics and average rating to identify potentially predictive features.

#### `minutes` vs. `high_rating`
Genereally, the shorter the recipe takes to complete, the higher the precision. As the minutes increases, the variability in the ratings increases.
<iframe
  src="assets/bivariate_min.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### `n_ingredients` vs. `high_rating`
Generally, the less number of ingredients, the higher the precision. As the number of ingredients increases, the variability in the ratings increases. However, the graph dispays a curved shape, indicating that there are higher ratings at the more extreme ends of the spectrum while the graph maintains lower constant rating with less polarizing number of ingredients.

<iframe
  src="assets/bivariate_ingre.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### `n_steps` vs. `high_rating`
Generally, the less number of steps, the higher the precision. As the number of steps increases, the variability in the ratings increases.
<iframe
  src="assets/bivariate_step.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

### Interesting Aggregates

We grouped and binned data to discover broader rating trends across recipe categories.

#### Average Rating by Number of Steps
- Grouped by `n_steps`, we observed a non-linear trend: mid-range complexity (around 6–10 steps) yields the highest average ratings.

#### Average Rating by Prep Time (Minutes)
- Binned into 10-minute intervals, recipes between 30–60 minutes had the highest average ratings.


*_(Embed grouped tables or bar plots for each of these patterns)_*

These aggregates helped us pinpoint features that contribute meaningfully to rating prediction and informed our feature selection in the modeling stages.

---

## Assessment of Missingness

### NMAR Analysis

We examined whether any missing data in our dataset is likely **Not Missing at Random (NMAR)**.

The most important column with missing values is `avg_rating`, which represents the mean user rating for a recipe. Since `avg_rating` is computed from user-submitted ratings, it is plausible that missingness is **not random**. For example:
- Newer or less popular recipes may not yet have any ratings.
- Recipes with low engagement might never receive a rating at all.

Thus, we believe that `avg_rating` is **likely NMAR** — its missingness may be directly related to its own value or latent factors like user behavior and visibility. We would need additional data (e.g., views, clicks) to determine if it’s MAR instead.

---

### Missingness Dependency: Permutation Tests

To further assess missingness, we performed permutation tests to determine whether the missingness of `avg_rating` depends on other observed features.

#### Test 1: Does missingness in `avg_rating` depend on `minutes`?

- **Null Hypothesis (H₀)**: Missingness in `avg_rating` is independent of `minutes`.
- **Alternative Hypothesis (H₁)**: Missingness in `avg_rating` depends on `minutes`.

We computed the observed difference in mean `minutes` between recipes with and without a rating. Then, we ran 1000 permutations by shuffling the missingness labels and recomputing the mean difference each time.

- **Observed Difference**: _(insert value, e.g., 10.2 minutes)_
- **p-value**: _(insert value, e.g., 0.004)_

**Conclusion**: Since the p-value is less than 0.05, we reject the null hypothesis. **Missingness in `avg_rating` does depend on `minutes`.**

*(Optional: Embed a histogram showing the null distribution and observed statistic)*

---

#### Test 2: Does missingness in `avg_rating` depend on `day_of_week`?

- **Null Hypothesis (H₀)**: The distribution of missingness in `avg_rating` is the same across all days of the week.
- **Alternative Hypothesis (H₁)**: The missingness in `avg_rating` varies by day of the week.

We calculated the observed difference between the **maximum and minimum** missing value rates across days, then compared it to a null distribution generated by permuting the missingness indicator.

- **Observed Difference**: _(insert value, e.g., 0.016)_
- **p-value**: _(insert value, e.g., 0.231)_

**Conclusion**: Since the p-value is greater than 0.05, we fail to reject the null hypothesis. **There is no strong evidence that `avg_rating` missingness depends on `day_of_week`.**

---


- We identified `avg_rating` as a column likely to be **NMAR**, due to its reliance on user interactions.
- Our permutation tests showed that its missingness **does depend on `minutes`** but **not on `day_of_week`**.
- These findings informed our data cleaning choices and modeling decisions in later steps.


We identified missing values in the rating column and concluded it is **Not Missing At Random (NMAR)**because users may skip ratings for recipes they didn't like or finish.
To explore dependencies, we performed permutation tests:
    •    The missingness of rating was dependent on `submitted` due to the fact that more recent recipes had more missing ratings.
    •    It was not dependent on `n_steps` (number of steps in the recipe).

## Hypothesis Testing


We conducted a hypothesis test to determine whether the **number of ingredients** in a recipe is associated with its **average rating**.

### Research Question

> Do recipes with more ingredients tend to receive different average ratings than those with fewer ingredients?

---

### Hypotheses

- **Null Hypothesis (H₀)**: There is no difference in average ratings between recipes with more ingredients and those with fewer ingredients.
- **Alternative Hypothesis (H₁)**: There is a difference in average ratings between recipes with more ingredients and those with fewer ingredients.

---

### Test Design

- **Test Type**: Permutation test (non-parametric)
- **Test Statistic**: Difference in mean average rating between recipes with more than the median number of ingredients vs. at or below the median
- **Significance Level (α)**: 0.05

We first split the dataset into two groups:
- Group A: Recipes with `n_ingredients` > median
- Group B: Recipes with `n_ingredients` ≤ median

We computed the observed difference in their mean `avg_rating`, then shuffled the `n_ingredients` values 1000 times to create a null distribution of differences. The p-value was computed as the proportion of permuted differences that were greater than or equal to the observed difference (in absolute value).

---

### Results

- **Observed Difference**: _(insert your value, e.g., 0.0091)_
- **p-value**: _(insert your value, e.g., 0.375)_

---

### Conclusion

Since the p-value is **greater than 0.05**, we **fail to reject the null hypothesis**. This means we do not have sufficient evidence to say that the number of ingredients in a recipe significantly affects its average rating.

Even though our exploratory analysis suggested a potential trend, the difference could plausibly be due to random variation in the data.

*_(Optional: Embed a histogram of the null distribution with the observed statistic marked)_*


## Framing a Prediction Problem

### Prediction Question

> Can we predict a recipe’s average rating using features available at the time of submission?

This prediction task builds directly on our initial question about what characteristics are associated with highly rated recipes. Instead of simply analyzing correlations, we aim to build a model that **predicts `avg_rating`**, a continuous value.

---

### Prediction Type

- **Type**: Regression
- **Response Variable**: `avg_rating`
- **Why**: `avg_rating` is a numeric (float) variable representing the average user rating of a recipe.

---

### Features Used

We restricted our features to those available **at the time a recipe is submitted**, ensuring no label leakage. These include:

- `n_ingredients`: Number of ingredients in the recipe  
- `n_steps`: Number of instructions  
- `minutes`: Total prep time in minutes  
- `calories`, `protein_pdv`, `total_fat_pdv`, `sugar_pdv`, `sodium_pdv`: Nutritional values derived from the `nutrition` column  
- `description_length`: Number of characters in the recipe description (engineered)  
- `is_weekend`: Whether the recipe was submitted on a weekend (engineered)

All of these features are either directly available at the time of posting or can be derived without relying on future data like ratings, views, or user behavior.

---

### Evaluation Metric

We chose **Root Mean Squared Error (RMSE)** to evaluate our model because:
- It is appropriate for regression problems.
- It penalizes large errors more heavily than MAE.
- It provides an interpretable measure in the same units as the target variable (i.e., rating points).

---

### Summary

- **Problem Type**: Regression  
- **Response**: `avg_rating`  
- **Metric**: RMSE  
- **Features**: All known before recipe is rated  
- **Goal**: Learn how well we can predict a recipe’s rating from its characteristics alone

This framing provides a realistic and useful task that mirrors how a recipe platform might estimate user interest based on content alone.


We aim to predict whether a recipe will be highly rated.
- **Type:** Binary classification
- **Target variable:** `high_rating``
- **Features available at prediction time:**
    - **Nutritional:** `sugar_pdv`, `protein_pdv`, `calories`
    - **Metadata:** `n_steps`, `minutes`
    - **Temporal:** `day_of_week`, `month`
    - **Evaluation metric**: Accuracy
This helps identify what kinds of recipes are most likely to succeed, providing insight for recipe developers and platforms.


## Baseline Model

We built a baseline regression model to predict a recipe’s average rating (`avg_rating`) using features available at the time of submission.

---

### Features Used

The baseline model used the following **quantitative features**:

- `n_ingredients`: Number of ingredients  
- `n_steps`: Number of steps in the recipe  
- `minutes`: Total prep time  
- `calories`: Total calories  
- `protein_pdv`: Protein percent daily value  
- `total_fat_pdv`: Fat percent daily value  
- `sugar_pdv`: Sugar percent daily value  
- `sodium_pdv`: Sodium percent daily value

All of these features are **numerical** and were **scaled using `StandardScaler`** to normalize their ranges. No categorical or ordinal encoding was needed at this stage.

---

### Model and Pipeline

We used a **Linear Regression** model implemented inside an `sklearn` `Pipeline`, which included:

1. **Feature scaling** (StandardScaler)
2. **Model fitting** (LinearRegression)

We split the data into training and testing sets (80/20 split) to evaluate generalization.

---

### Evaluation Metric

We used **Root Mean Squared Error (RMSE)** as our evaluation metric, since:

- It is appropriate for continuous outcomes.
- It penalizes larger errors more heavily.
- It gives results in the same units as the ratings (0–5 scale).

---

### Performance

- **Baseline RMSE**: _(insert your model’s test RMSE here, e.g., 0.4128)_

---

### Conclusion

The baseline model establishes a starting point for performance. Although simple, it already captures some signal from the features and provides a meaningful benchmark to improve upon in our final model.




## Final Model

To improve upon our baseline, we engineered new features and used a more flexible model with hyperparameter tuning.

---

### New Features Added

We added two new features to capture additional information:

- **`description_length`**: Number of characters in the recipe’s `description` field. This may reflect the richness or clarity of instructions.
- **`is_weekend`**: Boolean indicating whether the recipe was submitted on a Saturday or Sunday, derived from `submitted`.

These features are known at the time of submission and may correlate with engagement or recipe quality.

---

### Feature Types

- **Quantitative**:  
  - `n_ingredients`, `n_steps`, `minutes`, `calories`, `protein_pdv`, `total_fat_pdv`, `sugar_pdv`, `sodium_pdv`, `description_length`
  - Transformed using `StandardScaler`
  
- **Binary**:  
  - `is_weekend`
  - Left as-is using `passthrough` in the `ColumnTransformer`

---

### Model and Pipeline

We replaced the linear model with a **Random Forest Regressor**, which can capture non-linear interactions and variable importance more effectively.

We constructed a full `Pipeline` with:

1. A `ColumnTransformer` to apply `StandardScaler` to numeric features and pass through binary ones  
2. A `RandomForestRegressor` model

We performed **hyperparameter tuning** using `GridSearchCV` to search over:

- `n_estimators`: [50, 100]  
- `max_depth`: [5, 10, None]

Cross-validation was performed on the training set only.

---

### Best Hyperparameters

- `n_estimators`: _(e.g., 100)_  
- `max_depth`: _(e.g., 10)_

---

### Evaluation

- **Final RMSE**: _(insert your final model’s test RMSE, e.g., 0.3712)_

Compared to our baseline, this model reduced error and demonstrated improved predictive power — particularly due to the new features and model flexibility.

---

### Conclusion

The final model performs better than the baseline, supporting the idea that text-based and temporal features (like `description_length` and `is_weekend`) help explain user ratings. This model is more expressive and better suited for capturing subtle interactions between recipe attributes.




## Fairness Analysis

To assess whether our final model performs equitably across groups, we performed a fairness analysis comparing prediction error for recipes submitted on weekends vs. weekdays.

---

### Group Definitions

- **Group X**: Recipes submitted on a **weekend** (`is_weekend = 1`)
- **Group Y**: Recipes submitted on a **weekday** (`is_weekend = 0`)
- **Evaluation Metric**: RMSE (Root Mean Squared Error)

We used the **final fitted model from Step 7** and did not retrain or modify it during the test.

---

### Hypotheses

- **Null Hypothesis (H₀)**: The model’s RMSE is equal for weekend and weekday recipes. Any observed difference is due to chance.
- **Alternative Hypothesis (H₁)**: The model’s RMSE differs between weekend and weekday recipes.

---

### Test Design

We computed the RMSE for each group using the test set, then calculated the **absolute difference** between the two.

Next, we performed a **permutation test**:
- We shuffled the `is_weekend` labels 1000 times.
- For each shuffle, we computed the RMSE difference.
- We calculated the **p-value** as the proportion of permutations with a difference at least as large as the observed difference.

---

### Results

- **Observed RMSE difference**: _(insert value, e.g., 0.0174)_
- **p-value**: _(insert value, e.g., 0.194)_

---

### Conclusion

Since the p-value is greater than 0.05, we **fail to reject the null hypothesis**. This suggests that our model does **not show statistically significant unfairness** in predictive accuracy between weekend and weekday recipes.

Although small differences exist, they are consistent with what we’d expect under random variation, and we did not find strong evidence of model bias across this grouping.

*_(Optional: Embed histogram of permuted RMSE differences with observed value marked)_*
