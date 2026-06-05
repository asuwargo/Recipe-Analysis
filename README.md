# Recipe Analysis: Calorie Prediction and the Effect of Cooking Time on Ratings

## Introduction
Food.com is a popular website where people can submit, review, and look up different recipes. 
This dataset is a subset of Food.com because of how large the real dataset is. 

The first dataset contains 731,927 rows, where each row contains a review on a recipe. 
The columns it includes are:

| Column | Description |
|---|---|
| `'user_id'` | User ID |
| `'recipe_id'` | Recipe ID |
| `'date'` | Date of interaction |
| `'rating'` | Rating given |
| `'review'` | Review text |

The second dataset contains 83,782 rows, where each row contains a unique recipe. 
The columns it includes are:

| Column | Description |
|---|---|
| `'name'` | Recipe name |
| `'id'` | Recipe ID |
| `'minutes'` | Minutes to prepare recipe |
| `'contributor_id'` | User ID who submitted this recipe |
| `'submitted'` | Date recipe was submitted |
| `'tags'` | Food.com tags for recipe |
| `'nutrition'` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| `'n_steps'` | Number of steps in recipe |
| `'steps'` | Text for recipe steps, in order |
| `'description'` | User-provided description |

The central question of this project is: **Is there a relationship between cooking time and the average rating of a recipe?** I want to understand whether cooking time affects how people rate a recipe.

I also aim to **predict the calories in a recipe without using nutritional values**, which is helpful for people who are health conscious or have specific dietary goals.

## Data Cleaning and Exploratory Data Analysis 
The following data cleaning steps were performed:

1. The `'recipes'` and `'interactions'` datasets were merged on `'id'` and `'recipe_id'` using a left join. A left join was used to keep all recipes even if they had no reviews, since we want to analyze all recipes regardless of whether they received any ratings.

2. Ratings of 0 were replaced with `NaN` because Food.com does not allow users to submit a rating of 0. A 0 likely means the user left a review 
without a rating. Keeping 0s would have artificially lowered the average rating for those recipes and skewed the analysis.

3. The average rating per recipe was computed by grouping by `'id'` and taking the mean of `'rating'`, then merged back into the `'recipes'` 
dataframe. This gives each recipe a single representative rating instead of multiple rows for each review.

4. The `'nutrition'` column was stored as a string that looked like a list, so it was parsed and split into separate numeric columns: `'calories'`, `'total_fat'`, `'sugar'`, `'sodium'`, `'protein'`, `'saturated_fat'`, and `'carbohydrates'`. This made the nutritional values accessible for analysis and modeling.

5. The `'submitted'` and `'date'` columns were converted from strings to datetime objects. This allows for time-based analysis such as grouping 
recipes by year and comparing model performance across different time periods.

After cleaning the data, these are the columns and the data type of each column: 
| Column | Description |
|:---|:---|
| `'name'` | object |
| `'id'` | int64 |
| `'minutes'` | int64 |
| `'contributor_id'` | int64 |
| `'submitted'` | datetime64[ns] |
| `'tags'` | object |
| `'nutrition'` | object |
| `'n_steps'` | int64 |
| `'steps'` | object |
| `'description'` | object |
| `'ingredients'` | object |
| `'n_ingredients'` | int64 |
| `'rating'` | float64 |
| `'calories'` | float64 |
| `'total_fat'` | float64 |
| `'sugar'` | float64 |
| `'sodium'` | float64 |
| `'protein'` | float64 |
| `'saturated_fat'` | float64 |
| `'carbohydrates'` | float64 |

Since there are a lot of columns in the dataframe, here is the head of the cleaned dataframe with only several columns that are relevant:
| name                                 |   minutes |   n_steps |   n_ingredients |   rating |   calories |
|:-------------------------------------|----------:|----------:|----------------:|---------:|-----------:|
| 1 brownies in the world    best ever |        40 |        10 |               9 |        4 |      138.4 |
| 1 in canada chocolate chip cookies   |        45 |        12 |              11 |        5 |      595.1 |
| 412 broccoli casserole               |        40 |         6 |               9 |        5 |      194.8 |
| millionaire pound cake               |       120 |         7 |               7 |        5 |      878.3 |
| 2000 meatloaf                        |        90 |        17 |              13 |        5 |      267   |

### Univariate Analysis
<iframe
  src="assets/minutes-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The distribution of cooking time is right-skewed, with most recipes taking under 60 minutes to prepare. This suggests that the majority of recipes on Food.com are relatively quick to make.
<iframe
  src="assets/calories-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The distribution of calories is also right-skewed, with most recipes containing under 500 calories. A small number of recipes have very high calorie counts, which is why outliers were removed for visualization purposes.

### Bivariate Analysis
<iframe
  src="assets/minutes-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The scatter plot of cooking time vs average rating shows no clear relationship between the two variables. Recipes of all cooking times tend to receive similar ratings, suggesting that cooking time alone does not strongly influence how a recipe is rated.
<iframe
  src="assets/nsteps-calories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The scatter plot of number of steps vs calories shows a slight positive trend — recipes with more steps tend to have slightly higher calories. This makes sense as more complex recipes likely use more ingredients and result in higher calorie dishes.

## Interesting Aggregates
The table below shows the average calories by cooking time range:
| time_range   |   calories |
|:-------------|-----------:|
| 0-30         |    341.89  |
| 30-60        |    428.849 |
| 60-120       |    528.47  |
| 120-300      |    537.993 |

Recipes that take longer to cook tend to have slightly higher average calories. This suggests that longer recipes are generally more complex and calorie-dense than quick recipes

## Assessment of Missingness

### NMAR Analysis
I believe the `'rating'` column is NMAR because people who don't really care about the recipe and doesn't have any significant experience that will lead them to leave a rating, will not rate at all. So most people who had a not so good experience with the recipe will most likely not leave a rating compared to someone who had a good experience, they will more likely leave a rating. 

### Missingness Dependency
To examine the missingness of `'description'`, permutation test is done to determine whether or not the column is dependent on another column.
I tested description on 2 different columns `'minutes'` and `'ingredients'`.

**Does description missingness depend on `'minutes'`?**

- **Null Hypothesis:** The missingness of `'description'` does not depend on `'minutes'`. Any difference in means is due to random chance.
- **Alternative Hypothesis:** The missingness of `'description'` does depend on `'minutes'`.
- **Test Statistic:** Absolute difference in means of `'minutes'` between recipes with and without a description.
- **Result:** The observed difference in means was 0.584 and the p-value was 0.59, which is above the 0.05 significance level. We fail to reject the null hypothesis — the missingness of `'description'` does not depend on `'minutes'`.

<iframe
  src="assets/missing-minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Does description missingness depend on `'n_ingredients'`?**

- **Null Hypothesis:** The missingness of `'description'` does not depend on `'n_ingredients'`. Any difference in means is due to random chance.
- **Alternative Hypothesis:** The missingness of `'description'` does depend on `'n_ingredients'`.
- **Test Statistic:** Absolute difference in means of `'n_ingredients'` between recipes with and without a description.
- **Result:** The p-value was 0.001, which is below the 0.05 significance level. We reject the null hypothesis — the missingness of `'description'` does depend on `'n_ingredients'`.

<iframe
  src="assets/missing-n-ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



