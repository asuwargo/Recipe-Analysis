# Recipe Analysis: Calorie Prediction and the Effect of Cooking Time on Ratings

## Introduction
Food.com is a popular website where people can submit, review, and look up different recipes. 
This dataset is a subset of Food.com because of how large the real dataset is. 

The first dataset `inter` contains 731,927 rows, where each row contains a review on a recipe. 
The columns it includes are:

| Column | Description |
|---|---|
| `user_id` | User ID |
| `recipe_id` | Recipe ID |
| `date` | Date of interaction |
| `rating` | Rating given |
| `review` | Review text |

The second dataset `recipes` contains 83,782 rows, where each row contains a unique recipe. 
The columns it includes are:

| Column | Description |
|---|---|
| `name` | Recipe name |
| `id` | Recipe ID |
| `minutes` | Minutes to prepare recipe |
| `contributor_id` | User ID who submitted this recipe |
| `submitted` | Date recipe was submitted |
| `tags` | Food.com tags for recipe |
| `nutrition` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| `n_steps` | Number of steps in recipe |
| `steps` | Text for recipe steps, in order |
| `description` | User-provided description |

The central question of this project is: **Is there a relationship between cooking time and the average rating of a recipe?** I want to understand whether cooking time affects how people rate a recipe.

I also aim to **predict the calories in a recipe without using nutritional values**, which is helpful for people who are health conscious or have specific dietary goals. 
