# Do Sugary Recipes Get Higher Ratings in Certain Seasons?
Authors: Audrey Chung & Amrutha Potluri

## Introduction

We explore the question: Does the season in which a recipe is submitted affect how sugary recipes are rated?
Using data from [Food.com](https://www.food.com/?ref=nav)
, we analyze how seasonal trends may influence user ratings, particularly for recipes high in sugar. This question matters because recipe creators and platforms might benefit from understanding how timing impacts feedback. For instance, do users rate sugary desserts more generously in the winter holiday season?
Our dataset includes:
    •    83,782 recipes
    •    731,927 user ratings
Relevant columns we use:
    •    sugar_pdv: percent daily value of sugar in a recipe
    •    date: date the recipe was reviewed
    •    avg_rating: average user rating for the recipe
    •    month: derived from the submission date
    •    high_rating: whether the average rating is at least 4.5

## Data Cleaning and Exploratory Data Analysis

To prepare our data:
    •    We merged the recipes and ratings datasets.
    •    We replaced all ratings of 0 with NaN, treating them as missing since users likely left no rating.
    •    We computed avg_rating per recipe and created a binary high_rating column.
    •    We extracted month and day_of_week from the submission date.
    •    We parsed the nutrition column into separate columns like sugar_pdv, protein_pdv, and calories.
We then explored the distribution of sugar_pdv and examined whether high_rating proportions varied by month.

## Assessment of Missingness

We identified missing values in the rating column and concluded it is Not Missing At Random (NMAR) because users may skip ratings for recipes they didn't like or finish.
To explore dependencies, we performed permutation tests:
    •    The missingness of rating was dependent on submitted due to the fact that more recent recipes had more missing ratings.
    •    It was not dependent on n_steps (number of steps in the recipe).

## Hypothesis Testing

Hypothesis:
    •    Null (H₀): Season does not affect the rating of sugary recipes.
    •    Alternative (H₁): Season does affect the rating of sugary recipes.
We filtered recipes with a sugar daily value of 20% or more and created a season column from the month of submission. Using the Total Variation Distance (TVD) between high_rating distributions across seasons, we measured how much these distributions differed.
After 1,000 permutations where the season column was shuffled, we found:
    •    Observed TVD: 0.01587
    •    p-value: < 0.001
This provides strong evidence that the season does affect how sugary recipes are rated.

## Framing a Prediction Problem

We aim to predict whether a recipe will be highly rated.
    •    Type: Binary classification
    •    Target variable: high_rating
    •    Features available at prediction time:
    ◦    Nutritional: sugar_pdv, protein_pdv, calories
    ◦    Metadata: n_steps, minutes
    ◦    Temporal: day_of_week, month
    •    Evaluation metric: Accuracy
This helps identify what kinds of recipes are most likely to succeed, providing insight for recipe developers and platforms.

## Baseline Model

We trained a logistic regression classifier using a simple pipeline with:
    •    Numeric features: sugar_pdv, protein_pdv
    •    Categorical features: month, day_of_week
Categorical variables were encoded using OneHotEncoder. All preprocessing and model training were done using a single Pipeline.
    •    Accuracy: 0.7274
This model serves as a benchmark for future improvement.

## Final Model



## Fairness Analysis