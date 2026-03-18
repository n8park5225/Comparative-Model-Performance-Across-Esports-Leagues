# Comparative Model Performance Across Esports Leagues

This project was completed as the final project for DSC 80 (Practice and Application of Data Science).

---

## Introduction

### Understanding the Dataset

This project uses a League of Legends esports match statistics dataset spanning multiple years from 2014 to 2026. The dataset contains both player-level observations and team-level summaries for each match played across various professional leagues.

Since this project focuses on predicting match outcomes, the analysis will be conducted using team-level rows only, where each row represents the performance of one team in a single match. These rows contain a win/loss indicator that will serve as the response variable for prediction.

### Potential Research Questions

- Does the number of kills a team secures correlate with their probability of winning?
- Is gold advantage associated with match outcome?
- Are teams that secure more objectives (towers, dragons) more likely to win?
- Do certain leagues or regions exhibit higher win rates?
- Which combination of performance metrics best predicts whether a team will win a match?

### The Research Question

Which combination of performance metrics best predicts whether a team will win a match?

### Why This Matters

Esports teams and analysts often seek to understand which measurable factors are most strongly associated with winning. Identifying the combination of performance metrics that best predicts match outcomes can help inform strategy, coaching decisions, and post-match analysis through data-driven insights.

### Relevant Columns

- **result**: Indicates whether a team won (1) or lost (0)  
- **kills, deaths, assists**: Team combat performance  
- **goldearned**: Total team gold  
- **towers, inhibitors**: Structure objectives secured  
- **dragons, barons**: Neutral objectives secured  
- **visionscore**: Team vision control  

Total number of rows in team-level dataset: ________  
Number of relevant columns used for prediction: ________  

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The original dataset contains both player level and team level rows for each match. Since the research question focuses on predicting whether a team wins a match based on performance metrics, the dataset was filtered to include only team level observations. Relevant columns such as kills, deaths, assists, goldat15, and gamelength were retained for analysis.

### Univariate Analysis

We first examined the distribution of key performance metrics to better understand their variability across matches. The distribution of game length showed that most professional matches tend to fall within a consistent duration range, indicating relatively standardized pacing across competitive play.

### Bivariate Analysis

To explore relationships between performance metrics and match outcomes, we compared team kills and early gold advantage against match results. Teams that won matches generally had higher kill counts and greater gold advantages at the 15 minute mark compared to teams that lost.

![Kills by Match Result](assets/bivariate_kills_result.png)

![Gold at 15 Minutes by Match Result](assets/bivariate_gold15_result.png)

### Interesting Aggregates

We also grouped the data by league to compute win rates and total number of games played. This helps identify whether performance trends are consistent across different competitive regions.

![League Summary](assets/league_summary.png)

---

## Assessment of Missingness

I believe **visionscore** may be **Missing Not At Random (MNAR)**. Vision score depends on warding and other vision actions that are driven by team strategy and in-game performance. If a team is far behind, they may place fewer wards and take fewer vision-related actions, which can lead to a lower or missing vision score. This means the probability that **visionscore** is missing may depend on the true (unobserved) value of visionscore itself.

To better explain this missingness and potentially make it closer to MAR, it would help to have additional data such as team gold difference over time, map control indicators, or a timeline of ward placements. These variables would capture game state and team behavior that could explain why vision activity is missing.

To study whether the missingness of visionscore depends on other variables, I created an indicator column **visionscore_missing**, which is True when visionscore is missing and False otherwise. I then compared game outcomes and performance metrics between these two groups.

The plot below compares the distribution of kills when visionscore is missing versus not missing. Differences in these distributions suggest that missingness is related to in-game performance.

![Missingness Analysis](assets/missingness_kills.png)

---

## Hypothesis Testing

We test whether early game gold advantage is associated with match outcome.

- **Null Hypothesis (H₀):** The average gold at 15 minutes is the same for winning teams and losing teams.  
- **Alternative Hypothesis (H₁):** Winning teams have higher average gold at 15 minutes than losing teams.  

**Test Statistic:** Difference in mean gold at 15 minutes between winning and losing teams  

A permutation test was conducted using 1000 permutations with significance level α = 0.05.

**p-value:** ______  

Since this p-value is less than 0.05, we reject the null hypothesis and conclude that winning teams tend to have higher gold at 15 minutes.

![Permutation Distribution](assets/step4_permutation.png)

---

## Framing a Prediction Problem

### Prediction Problem

The goal of this project is to predict whether a team will win a professional League of Legends match based on team-level performance metrics recorded during the game.

### Response Variable

- **result = 1:** Win  
- **result = 0:** Loss  

### Problem Type

Binary classification.

### Features Used

- kills, deaths, assists  
- goldearned  
- towers, inhibitors  
- dragons, barons  
- visionscore  

### Evaluation Metric

Accuracy and F1-score.

### Time of Prediction

All features are available by the end of a match and do not include the outcome itself.

---

## Baseline Model

The baseline model is a Logistic Regression classifier using:

- kills  
- goldspent  

**Performance:**

- Accuracy: 0.8222  
- F1-score: 0.8220  

---

## Final Model

The final model uses engineered features:

- kill_diff  
- gold_per_kill  

and a Random Forest classifier with tuned hyperparameters.

**Performance:**

- Accuracy: 0.9553  
- F1-score: 0.9554  

The final model substantially improves upon the baseline model, suggesting that engineered features better capture meaningful in-game advantages and that the Random Forest model effectively models nonlinear relationships.

---

## Fairness Analysis

### Fairness Question

Does the model perform differently for teams in the LCK versus teams in other leagues?

### Groups

- **Group X:** LCK teams  
- **Group Y:** Non-LCK teams  

### Evaluation Metric

F1-score.

### Hypotheses

- **Null Hypothesis (H₀):** Model performance is equal across groups  
- **Alternative Hypothesis (H₁):** Model performance differs  

### Test Statistic

F1(LCK) − F1(non-LCK)

### Method

A permutation test was conducted by shuffling league labels and recomputing the F1-score difference.

### Results

- F1(LCK): LCK_F1  
- F1(non-LCK): NON_LCK_F1  
- Observed difference: OBS_DIFF  
- p-value: P_VALUE  

### Conclusion

Since the p-value is greater than 0.05, we fail to reject the null hypothesis. There is no strong evidence that the model performs differently across these groups.