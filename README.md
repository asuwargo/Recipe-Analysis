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
| ----------- | ----------- |
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
|:---|---:|---:|---:|---:|---:|
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
| ----------- | ----------- |
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
- **Result:** The p-value was 0.551, which is above the 0.05 significance level. We fail to reject the null hypothesis — the missingness of `'description'` does not depend on `'minutes'`.

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

## Hypothesis Testing
To answer the question of whether or not there is a relationship between cooking time and the average rating of a recipe, hypothesis testing is conducted. 

**Null Hypothesis:** There is no relationship between cooking time (`'minutes'`) 
and the average rating of a recipe. Any observed correlation is due to random chance.

**Alternative Hypothesis:** There is a relationship between cooking time and 
the average rating of a recipe.

**Test Statistic:** Pearson correlation coefficient between `'minutes'` and `'rating'`.

**Significance Level:** 0.05

**Result:** The observed correlation was 0.0014 and the p-value was 0.688, which 
is well above the 0.05 significance level. We fail to reject the null hypothesis 
— there is no significant relationship between cooking time and average rating.

## Framing a Prediction Problem
The prediction problem is to **predict the number of calories in a recipe** 
given information about the recipe itself.

**Type:** Regression — calories is a continuous numeric variable.

**Response Variable:** `'calories'` — chosen because it is one of the most 
practically useful nutritional facts about a recipe. Many people who are health 
conscious or have specific dietary goals need to know the calorie content of a 
recipe. 

**Metric:** RMSE (Root Mean Squared Error) — chosen because it is in the same 
units as calories, making it interpretable. It also penalizes large errors more 
heavily than small ones, which is important here since being off by 1000 calories 
is much worse than being off by 50.

## Baseline Model
For the baseline model, the data is split into training data and test data with a test size of 0.2. Linear regression is used with `'n_ingredients'` and `'n_steps'` as the features, both are quantitative nominal values so there is no need to encode the values. RMSE is used as the metric and the result for the test set is **583.4**.

I think this model is not good because the RMSE is pretty high. The model is too simple and the features used are not enough to predict calories. 

## Final Model 

**Features added:**
1. One hot encoded `'tags'` (nominal)
Tags such as "dessert", "low-carb", and "healthy" directly reflect the type and nutritional profile of a recipe. 
A recipe tagged as "dessert" is likely to have more sugar and calories than one tagged as "low-carb".The top 20 most common tags were selected using `MultiLabelBinarizer`, which converts the list of tags for each recipe into 20 
binary columns (1 if the recipe has that tag, 0 if not). This gives the model 
information about the category and dietary nature of a recipe, which is strongly 
related to calorie content.

2. **`'log(n_steps)'`** (quantitative) — The distribution of `'n_steps'` is 
right-skewed, with most recipes having few steps but some having many. A 
`FunctionTransformer` with `np.log1p` was applied to compress the large values 
and make the relationship with calories more linear, which helps Ridge regression 
fit the data better. `np.log1p` was used instead of `np.log` to handle any 
potential zero values.

In total, the final model uses 22 features:
- 2 quantitative (`'n_steps'`, `'n_ingredients'`) — passed through as-is
- 1 quantitative engineered (`'log_steps'`) — log transform of `'n_steps'`
- 20 nominal (one-hot encoded tags) — binary columns for each of the top 20 tags

No ordinal features were used.

**Modeling Algorithm:** Ridge Regression — chosen as an upgrade from Linear 
Regression because it adds a regularization penalty (L2) that shrinks large 
coefficients toward zero, preventing overfitting. This is especially useful 
when adding many binary tag features, since some tags may not be very 
informative and Ridge will naturally reduce their influence.

**Pipeline:** All steps were implemented in a single `sklearn` Pipeline:
1. The features (`'n_steps'`, `'n_ingredients'`, `'log_steps'`, and the 20 tag 
columns) were concatenated into a single dataframe before being passed into the pipeline.
2. The Ridge model was then fit on these features.

**Hyperparameter Tuning:** The `'alpha'` hyperparameter in Ridge controls the 
amount of regularization — a higher alpha means more shrinkage. `GridSearchCV` 
with 5-fold `KFold` cross validation was used to search over 
`[0.1, 1, 10, 100]`. Each combination was evaluated using RMSE and the best 
alpha found was **10**, meaning a moderate amount of regularization worked best.

<iframe
  src="assets/hyperparameter-tuning.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Performance:** The final model achieved an RMSE of **578.9** on the test set, 
compared to the baseline RMSE of **583.4**. While the improvement is modest, 
it does show that adding tag features and the log transform helped the model 
better predict calories. The small improvement is likely because `'n_steps'` 
and `'n_ingredients'` are weakly correlated with calories to begin with (0.14 
and 0.12 respectively), so even with better features the model struggles to 
predict calories accurately without nutritional information. The assignment 
intentionally avoids using nutritional columns to prevent data leakage, which 
limits how much the model can improve.

## Fairness Analysis

**Groups:**
- **Group X (Old recipes):** Recipes submitted before 2010
- **Group Y (New recipes):** Recipes submitted in 2010 or after

**Evaluation Metric:** RMSE — the same metric used to evaluate the final model.

**Null Hypothesis:** The model is fair. Its RMSE for old recipes and new recipes 
are roughly the same, and any difference is due to random chance.

**Alternative Hypothesis:** The model is unfair. Its RMSE for old recipes is 
different from its RMSE for new recipes.

**Test Statistic:** Difference in RMSE (old recipes − new recipes).

**Significance Level:** 0.05

**Results:**
- RMSE for old recipes (before 2010): **567.8**
- RMSE for new recipes (2010 or after): **597.6**
- Observed difference: **-29.8**
- p-value: **0.62**

**Conclusion:** Since the p-value of 0.614 is well above the 0.05 significance 
level, we fail to reject the null hypothesis. The difference in RMSE between 
old and new recipes is not statistically significant and is likely due to random 
chance. The model appears to perform equally well for both old and new recipes.

<iframe
  src="assets/fairness-analysis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



