# League_of_legends_analysis

Name: Jingwen Gu

## Introduction

The dataset is about **league of legends**, a game produced by riot. It has a wide coverage of data of league of legends pro grames raging from regional league game such as LCS and LPL, to global events such as MSI and Word finals, ranging from 2014 to 2025. The dataset include lots of detailed information about each game, such as result, or gold difference at each timestampes. The question for me around this dataset is: **Can we have some idea, or even predict the result of the game, based on the information that we have before the 15 min of the game?**

Readers should care about this dataset because league of legends is a famous game with a huge fanbase around the globe. People who love to play the game will care about the importance of the 15 min to the whole game, and the prediction, since it makes people better enjoy the game and watching pro players playing this game.

The working table contains **821,270 player-game rows** and **165 columns**. Here are some relevant columns.

- `golddiffat15` – team's gold difference at the 15-minute mark, a key economic indicator derived from Riot's timeline data that reflects early farming, kills, and objective value.
- `xpdiffat15` – experience difference at 15 minutes, capturing level advantages that translate into ability power and base stat leads.
- `csdiffat15` – creep score (minion kills) difference at 15 minutes, measuring laning phase dominance and farming efficiency.
- `killsat15` / `deathsat15` / `assistsat15` – combat statistics at the 15-minute mark, reflecting early skirmish and teamfight outcomes.
- `firstdragon` – referee flag (bool) indicating whether the team secured the first elemental dragon before 15 minutes, representing early bot-side objective control.
- `firstherald` – flag for capturing the first Rift Herald (typically available around 8-10 minutes), reflecting early top-side macro pressure.
- `result` – win/loss indicator for the team/side represented by the row, derived from Riot's match result feed (target variable for prediction).
- `side` – categorical (Blue/Red) telling which draft side the player belonged to, important because side affects objective access and map control.



## Data Cleaning and Exploratory Data Analysis

### Data Cleaning 

The dataset has one record per participant plus two team-level summary rows for every `gameid`, so I first confirmed each match contained 12 rows and then dropped the team summaries to avoid double-counting.  I restricted the table to `complete` games only by the row datacompletenes, which eliminates games which only stores significantly part of the game. During cleaning I found that Riot timeline deltas (`golddiff*`, `xpdiff*`, `csdiff*`) store -1 whenever the live parser times out; I replaced those -1 with `NaN` so aggregates ignore the missing stages rather than biasing the analysis downward, since the rows will be crucial to the analysis below. Event flags such as `firstdragon` and `firstbaron` originate from referee annotations and were stored as integers, so I cast them to Boolean to make filtering by objectives natural. Finally, I standardized time data by converting the text `date` column into an aware UTC timestamp (`match_start_utc`) and `gamelength` seconds into both a pandas `Timedelta` and minute duration. Then we deleted unused columns to make the dataframe clean.

Below is a snippet of our cleaned dataframe.

| gameid   | side | teamname | league | date                | gamelength | patch | golddiffat15 | xpdiffat15 | csdiffat15 | firstdragon | firstherald | firstbaron | playoffs | result | golddiffat20 | match_start_utc           | game_duration   | game_duration_minutes | patch_major | patch_minor |
| -------- | ---- | -------- | ------ | ------------------- | ---------- | ----- | ------------ | ---------- | ---------- | ----------- | ----------- | ---------- | -------- | ------ | ------------ | ------------------------- | --------------- | --------------------- | ----------- | ----------- |
| TRLH3/33 | Blue | Fnatic   | EU LCS | 2014-01-14 17:52:02 | 1924       | 3.15  | 49           | -560       | <NA>       | False       | <NA>        | True       | False    | True   | 386          | 2014-01-14 17:52:02+00:00 | 0 days 00:32:04 | 32.0667               | 3           | 15          |
| TRLH3/33 | Blue | Fnatic   | EU LCS | 2014-01-14 17:52:02 | 1924       | 3.15  | -1107        | -703       | -36.0      | False       | <NA>        | True       | False    | True   | -1204        | 2014-01-14 17:52:02+00:00 | 0 days 00:32:04 | 32.0667               | 3           | 15          |
| TRLH3/33 | Blue | Fnatic   | EU LCS | 2014-01-14 17:52:02 | 1924       | 3.15  | -145         | -148       | 7.0        | False       | <NA>        | True       | False    | True   | -118         | 2014-01-14 17:52:02+00:00 | 0 days 00:32:04 | 32.0667               | 3           | 15          |
| TRLH3/33 | Blue | Fnatic   | EU LCS | 2014-01-14 17:52:02 | 1924       | 3.15  | 1588         | 998        | 22.0       | False       | <NA>        | True       | False    | True   | 1900         | 2014-01-14 17:52:02+00:00 | 0 days 00:32:04 | 32.0667               | 3           | 15          |
| TRLH3/33 | Blue | Fnatic   | EU LCS | 2014-01-14 17:52:02 | 1924       | 3.15  | 887          | 476        | -15.0      | False       | <NA>        | True       | False    | True   | 1072         | 2014-01-14 17:52:02+00:00 | 0 days 00:32:04 | 32.0667               | 3           | 15          |

Univariate Analysis

<iframe
  src="assets/game_duration_hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot analyzes the distribution of game duration, which shows that most matches end between 25 and 40 minutes, with only a small tail beyond 45 minutes, giving us a good glimpse of the typical distribution of the game duration in league of legends. This trend means that marathon game and short game are all relatively rare events in league of legends.

### Bivariate Analysis

<iframe
  src="assets/objective_win_rates.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
Winning teams enjoy roughly a **10–20 percentage point boost** when they claim the first neutral objective.

### Interesting Aggregates

| league | Team-games — Did not secure | Team-games — Secured first dragon | Win rate — Did not secure | Win rate — Secured first dragon |
| ------ | --------------------------- | --------------------------------- | ------------------------- | ------------------------------- |
| LCK    | 4972                        | 4970                              | 0.387                     | 0.613                           |
| LCS    | 1654                        | 1654                              | 0.390                     | 0.610                           |
| LEC    | 1901                        | 1901                              | 0.410                     | 0.590                           |
| LPL    | 2263                        | 2263                              | 0.389                     | 0.611                           |
| MSI    | 615                         | 615                               | 0.367                     | 0.633                           |

We aggregated the games by league(we chose 5 significant league for better analysis since if we simply aggregate the league, there's going to be 100+ leagues including small competitions). It turns out that in every maojr league the win rate gets **much higher** if securing the first netural objective. 

## Assessment of Missingness

### NMAR analysis

I don't see evidence that any column in this dataset is NMAR. The only systematic missingness comes from the live scoreboard metrics at specific timestamps, such as golddiffat10, xpdiffat15, csdiffat20, caused by the API timeout. To explicitly make sure if it's MAR or NMAR, we may need the scraping process of these data, such as the per-match API error timestamp data. With such help, we can better make sure that the missing data are MAR instead of NMAR.

### Missingness Dependency

I performed two permutation test over here to analyze if the csdiffat20 column is dependent on other collumns. We show that the csdiffat20 is dependent on the **game duration**, while independent on the **side**.


- **Test 1 (duration dependence):**
  - `H0`: the missingness indicator `golddiffat20_missing` is independent of `game_duration_minutes`; any difference in mean duration between missing and observed rows is due to chance. While H1 means dependent.
  - **Statistic:** absolute difference in mean `game_duration_minutes` between missing and observed rows (`|Δ mean|`). Observed value ≈ 12.8 minutes.
  - **p-value:** ≈ 0.001 (1,000 permutations). Reject `H0`; short games are far more likely to miss the 20-minute delta, consistent with matches ending before the checkpoint or the parser timing out.

- **Test 2 (side independence):**
  - `H0`: `golddiffat20_missing` is independent of draft `side`; the crawler treats Blue and Red rows symmetrically. While H1 means dependent.
  - **Statistic:** absolute difference in the missingness rate between Blue and Red (`|Δ missing rate|`). Observed value ≈ 7×10⁻⁶.
  - **p-value:** ≈ 0.97 (1,000 permutations). Fail to reject `H0`; missingness does not depend on side.

<iframe
  src="assets/golddiffat20_missing_duration_perm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
This plot intuitively shows the gap of the actual change in duration caused by the missingness of golddiffat20 and the permuted difference if H0 in test1 is true, telling us we should reject H0 and the correlation betweeen the missingness of golddiffat20 and game duration actually exist.

## Hypothesis Testing

- **Hypotheses:** 
`H₀`: securing the first dragon does *not* change win rate (win outcomes are independent of the `firstdragon` flag). 
`H₁`: teams that secure the first dragon have a different win rate. We test the absolute difference in win proportions so the test is two-sided.
- **Test statistic:** `|Δ win rate| = |P(result=1 | firstdragon=True) - P(result=1 | firstdragon=False)|`. The observed statistic is ≈ **0.168** (66.7% vs 49.9% win rate).
- **Significance:** I set `α = 0.01` to demand very strong evidence before declaring an effect, which is appropriate because we have ~185k team-game rows and even tiny artifacts could otherwise appear “significant.”
- **p-value & conclusion:** The permutation distribution yields `p ≈ 0.0005 < α`, so we reject `H₀` and conclude that securing the first dragon is **strongly associated** with higher win rates. 
- **Why this test fits the research question:** Question asks whether early neutral objectives predict victory, while the test fits well with the question answering the question with an appropriate hypothesis test.

## Framing a Prediction Problem

**Classification problem**: predict whether a team wins by the available information before 15min(e.g., firstdragon, firstherald, golddiffat15, xpdiffat15, csdiffat15, side, playoffs, league, patch_major/minor, playername,teamname,position,champion/pick,ban). The column I'm trying to predict is **result**. It's a binary classification and the metric we use is accuracy (since the win and lose of result is balanced so we don't need F1-score). 

The information I know (I use to train the model) is just the available information before 15min. I did the 80/20 split in training and test set to make sure the information is mutual between baseline model and final model to make sure everything is fair.

## Baseline Model

For the baseline, I build a logistic regression classifier model which predicts whether a team wins or not using two features that are available before 15 min: 

- Quantitative (3): golddiffat15, xpdiffat15, csdiffat15 – continuous pre-15-minute gap stats passed straight through the pipeline. I did not scale or tweak this part of the feature.
- Ordinal/discrete numeric (2): patch_major, patch_minor – treated as numeric indicators of patch version. No extra encoding.
- Binary categorical (3): firstdragon, firstherald, playoffs – stored as  0/1 flags. No extra encoding.
- Nominal categorical (2): side, league – one-hot encoded (OneHotEncoder(handle_unknown='ignore')) to produce dummy variables that could be better used for logistic regression.

This is the performance: **Train accuracy: 0.621 Test accuracy: 0.619**. Honestly i think the prediction is good, at least fair, since I know it's impossible to have a ~0.8 accuracy, since the game will be no longer fun after 15 min, if we know about which team will win or lose after 15 minutes. Games such as league of legends thrive because of its possibility, especially at the late phase of the match.

## Final Model

I added two features for better for the precision task.

\- `objective_score = firstdragon + firstherald`: counts how many early neutral objectives a team secured before 15 minutes (0, 1, or 2). Treating these flags jointly captures the intuition that stacking dragon plus Herald yields a larger macro lead than either objective alone.

\- `macro_score = (golddiffat15 / 1000) + (xpdiffat15 / 500) + (csdiffat15 / 100)`: combines three tempo indicators into a single standardized number. The denominators place gold, XP, and CS differences on similar scales, summarizing which team controls the map by the 15-minute mark without introducing post-15 information.

Both of the new features are great indicators of the 'overall advantage'.

To get a higher boost in prediction, I decided to use XGBoost, Random Forest, and MLP to fit the data, run a GridSearchCV for the best hyperparameters for the training, choose the best hyperparameters respectively, and choose the model with the highest test accuracy.

Here are the train and test accuracy for the three models:

* **XGBoost**: Train accuracy: 0.674 Test accuracy: 0.665 

  {'colsample_bytree': 1.0, 'learning_rate': 0.05, 'max_depth': 3, 'n_estimators': 400, 'subsample': 0.8} 

* **Random forest**: Train accuracy: 0.685  Test accuracy: 0.661

  {'max_depth': 12, 'max_features': 'sqrt', 'min_samples_split': 2, 'n_estimators': 150}

* **Neural network**: Train accuracy: 0.677 Test accuracy: 0.665

  {'alpha': 0.0003, 'hidden_layer_sizes': (128, 64), 'learning_rate_init': 0.003}

I rank the three models firstly by test accuracy, and then if the test accuracy evens, I'll compare the train accuracy. So by this conparison, **MLP** with {'alpha': 0.0003, 'hidden_layer_sizes': (128, 64), 'learning_rate_init': 0.003} wins slightly.

There's a modest improvement on the final model compared to the base model as expected, and I'm satisfied with the improvement since ~0.67 accuracy is at the sweet spot. Not too high to make the game boring, now too low to make my work useless.

## Fairness Analysis


I want to assess whether the final MLP model performs differently for teams on **Blue side** versus **Red side**. This is an important fairness question because draft side is assigned before the game and shouldn't systematically bias predictions if the model is fair.

- **Group X (Blue side):** Teams assigned to the Blue draft side
- **Group Y (Red side):** Teams assigned to the Red draft side

**Evaluation metric:** **Precision** (of predicting wins). We use precision because it answers: "When the model predicts a team will win, how often is it correct?" If the model is fair, precision should be similar for both sides.

**Hypotheses:**
- **H₀ (Null):** The model is fair. Precision for Blue side and Red side are equal; any observed difference is due to random chance.
- **H₁ (Alternative):** The model is unfair. Precision differs between Blue and Red sides.

**Test statistic:** `|precision_Blue - precision_Red|` (absolute difference in precision)

**Significance level:** α = 0.05

**Result**: 

* Precision for Blue side: 0.6654 
* Precision for Red side: 0.6506 
* Observed |Δ precision|: 0.0148 
* Permutation test p-value: 0.0370

So we slightly reject H0, showing our prediction with MLP is slightly unfair, performing better in predicting in Blue side. 
