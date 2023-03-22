# Eat healthier: Predicting Calories for Recipes on [Food.com](https://www.food.com/) 🥗

&nbsp; 

## PART I: Problem Framing

Intaking a moderate amount of calories per day is a crucial aspect of keeping fit. With more and more attention drawn to health maintenance nowadays, it’s important to accurately measure daily calories intake and plan meals accordingly to individual health conditions. Therefore, we are going to build a model to predict the calories of recipes based on the dataset containing information about recipes on [Food.com](https://www.food.com/). Our exploratory data analysis on this dataset can be found [here](https://xgui17.github.io/Recipe-Analysis/ ). 

We chose the **calories** of a recipe (`calories (#)`) as our **response variable** in the sense that our model can predict the calories of a recipe even if it’s not provided and help people better estimate their calories intake depending on their recipe choices. 

After reviewing the cleaned dataset, we decided to build a **linear regression** model to predict the `calories (#)` of a recipe. Since intuitively we do not have access to the ratings at the time when we predict a recipe’s calories, we are not going to include `ratings` as a feature. As for metrics to evaluate our model, we decided to use both **RMSE** and **R-squared**. While R-squared is conveniently scaled between 0 and 1 which makes it easier to interpret, RMSE allows us to directly measure how much our predictions deviate, on average, from the actual values. Thus, we are going to use **RMSE** to make comparisons across different hyperparameters and **R-squared** to measure the general performance of the models. 

&nbsp; 


## PART II: Baseline Model
### Baseline Features and Model
Our baseline model is a linear regression model with two features – `total fat (PDV)` and `sugar (PDV)`, both of which are quantitative continuous variables. We’ve chosen these two features because they are positively associated with calories in common sense. Our baseline model used a pipeline in which the FunctionTransformer converted the two features to percentages in two decimal places before passing them to a linear regression model. To ensure the model’s generalized performance on unseen data, we’ve divided the dataset into **training** and **testing** sets and fitted the model only on train data.  

### Baseline Performance
The baseline model’s **RMSE** on the training and testing sets are **135.02** and **138.35** (rounded to two decimal places). The RMSEs are relatively close, which means the model does a good job on generalizing to unseen data. The model got a reasonably high **score (R-Squared)** of **0.78** (rounded to two decimal places) on both training and testing sets, which is pretty *good*. However, there are other reasonable features we can add to improve the model’s performance.

&nbsp; 

## PART III: Final Model

### Final Features and Model
To improve the baseline model, we included four new features – `n_ingredients`, `sf_cut`, `cbhd_cut` and `year_since_2007`– in the input data frame and compared the performances of different combinations of features to determine the final model. To ensure the model’s generalized performance on unseen data, we’ve divided the dataset into **training** and **testing** sets and fitted the model only on train data.  

To be specific, we firstly included the `n_ingredients` columns as it is from the cleaned dataset. This feature is included because intuitively a recipe with more ingredients would probably have higher calories. The second groups of features added are `sf_cut` and `cbhd_cut`, which are created by discretizing the `saturated fat (PDV)` and `carbohydrates (PDV)` into 4 equal-sized buckets respectively based on quartile. These two features are qualitative ordinal variables and are added to the linear regression model after encoded by OneHotEncoder. The last added feature is  `year_since_2007`, which is created by subtracting 2007 from the year in which the recipes are submitted. This feature is included because we assume that more high-calorie recipes are being developed as time goes on. The year 2007 was chosen because the earliest recipe in our dataset was submitted in 2008.  We then used a **5-fold cross validation** on the training set to determine the best **hyperparameter** of polynomial features to use on this variable, which is degree 1. 

### Best Pipeline
After conducting a **5-fold cross validation** on different combinations of features including the one in the baseline model and comparing RMSEs, the best set of features are `total fat (PDV)`and `sugar (PDV)` transformed into percentages, along with `sf_cut`, `cbhd_cut`and `n_ingredients`. We used these features to construct our final linear regression model. 

### Final Performance
The final model’s **RMSE** on the training and testing sets are **107.23** and **111.05** (rounded to two decimal places). The RMSEs are relatively close, which means the final model does a good job on generalizing to unseen data. 
The final model got a pretty high **score (R-Squared)** of **0.86** (rounded to two decimal places) on both training and testing sets. The lower RMSE and higher score of the final model together show an *improvement* on the predicting accuracy compared to the baseline model.

&nbsp; 

## PART IV: Fairness Analysis

Finally, we would like to conduct a fairness analysis of our model by answering this question:
- “Does our model perform worse for individuals in Group X than it does for individuals in Group Y?”

Here we’ve chosen Group X to be the *Simple* recipes that can be completed within 9 steps (`n_steps` < 9), and Group Y be the *Complicated* recipes that require no less than 9 steps (`n_steps` >= 9). This threshold of 9 is chosen based on the median. 
The evaluation metric we choose is **RMSE**, and the null and alternative hypotheses are listed as below:
- Null hypothesis (H0)：Our model is fair. Its **RMSE** for *Simple* and *Complicated* recipes are roughly the same, and any differences are due to random chance.
- Alternative Hypothesis (Ha): Our model is unfair. Its **RMSE** for *Simple* recipes are lower than the one for *Complicated* recipes.

The test statistic we use is the difference between **RMSE** (*Simple* - *Complicated*). The observed difference is -13.68. This is a one-sided test, and the p-value is much smaller than 0.05. Therefore, we have sufficient evidence to reject the null hypothesis at 0.05 significance level and conclude that our model to predict calories performs better on *Simple* recipes with less than 9 steps. 

<iframe src="assets/permutation_result.html" width=600 height=410 frameBorder=0></iframe>
