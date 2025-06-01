# Seasoned to Sweetness: Do Sugary Recipes Get Higher Ratings in Certain Seasons?
Authors: Audrey Chung & Amrutha Potluri

## Introduction

The **Recipes and Ratings** dataset offers a rich look into user-submitted recipes, including ingredients, preparation steps, nutritional content, and user feedback in the form of ratings. With over 200,000 recipes and nearly 1 million interactions, it allows us to explore patterns that influence how well a recipe is received.

In this project, we investigate the question:

> **What recipe characteristics are associated with higher user ratings?**

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


`interactions`, contains 731927 rows, with 3 columns that are relevant to our investigation.

| Column Name | Description                             |
| ----------- | --------------------------------------- |
| `recipe_id` | Recipe ID (connects to `id` in Recipes) |
| `date`      | Date of rating or review                |
| `rating`    | Rating given                            |


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

| **Column Name**  | **Description**                                                           |
|------------------|---------------------------------------------------------------------------|
| `avg_rating`     | The average user rating                                 |
| `high_rating`    | Whether the average rating is at least 4.5                                |
| `n_ingredients`  | Number of ingredients in the recipe                                       |
| `n_steps`        | Number of steps in the recipe's instructions                              |
| `minutes`        | Total time (in minutes) to prepare the recipe                             |
| `nutrition`      | A list of 7 nutrition facts (e.g., calories, fat, sugar, etc.)            |
| `calories`       | Number of calories in a recipe                                            |
| `total_fat_pdv`  | Percent daily value of total fats in a recipe                             |
| `sugar_pdv`      | Percent daily value of sugar in a recipe                                  |
| `sodium_pdv`     | Percent daily value of sodium in a recipe                                 |
| `protein_pdv`    | Percent daily value of protein in a recipe                                |
| `sat_fat_pdv`    | Percent daily value of saturated fats in a recipe                         |
| `carbs_pdv`      | Percent daily value of carbohydrates in a recipe                          |
| `submitted`      | Timestamp of when the recipe was submitted                                |
| `day_of_week`    | Day of the week the recipe was submitted (derived from `submitted`)       |
| `is_weekend`     | Boolean indicating if it was submitted on a weekend (Sat/Sun)             |


### Univariate Analysis

<iframe
  src="assets/avg_rating_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

### Interesting Aggregates

## Assessment of Missingness

We identified missing values in the rating column and concluded it is **Not Missing At Random (NMAR)**because users may skip ratings for recipes they didn't like or finish.
To explore dependencies, we performed permutation tests:
    •    The missingness of rating was dependent on `submitted` due to the fact that more recent recipes had more missing ratings.
    •    It was not dependent on `n_steps` (number of steps in the recipe).

## Hypothesis Testing

**Hypothesis:**
- **Null (H₀):** Season does not affect the rating of sugary recipes.
- **Alternative (H₁):** Season does affect the rating of sugary recipes.
We filtered recipes with a sugar daily value of 20% or more and created a season column from the month of submission. Using the Total Variation Distance (TVD) between high_rating distributions across seasons, we measured how much these distributions differed.
After 1,000 permutations where the season column was shuffled, we found:
- **Observed TVD:** 0.01587
- **p-value:** < 0.001
Because we obtained a p-value of less than the significance level of 0.05, we reject the null as this provides strong evidence that the season does affect how sugary recipes are rated.

## Framing a Prediction Problem

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

We trained a logistic regression classifier using a simple pipeline with:
- **Numeric features:** `sugar_pdv`, `protein_pdv`
- **Categorical features:** `month`, `day_of_week`
Categorical variables were encoded using OneHotEncoder. All preprocessing and model training were done using a single Pipeline.
- **Accuracy:** 0.7274
This model serves as a benchmark for future improvement.

## Final Model



## Fairness Analysis