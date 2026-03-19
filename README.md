# Win or Lose — League of Legends

**By Chaylen Nguyen-Tran and Jeffrey Kang**

---

## Introduction

League of Legends is a popular PC video game released October 27, 2009 under Riot Games. League of Legends, commonly referred to as "League" or "LoL" for short is a Multiplayer Online Battle Area (MOBA) where 2 teams of 5 where each player controls one of the 172 playable champions (as of March 2026). There are multiple layers or in other words "objectives" each player does in order to win the game such as obtaining kills, killing neutral monsters, destroying enemy towers, and overall invading the other team's base and destroying their enemy's nexus. However, sometimes individual statistics are not enough to predict the game outcome, and individual player statistics need to be compared to the enemy team to see how much deficit or advanced they are, and how good the individual player is doing compared to their own team. League remains one of the top most played PC games with 150+ million registered players. League of Legends has a worldly loved competitive scene commonly referred to as "Pro Play" where people around the globe battle in this online area for money and worldly recognition.

Game statistics, as listed above, are carefully watched and followed in order to enhance the likelihood of a team winning in Pro Play. But, the question remains: How do these game statistics interact with each other, and which ones are more important, in respect to winning a game of League of Legends.

In this project, we are answering the question of how different players' game performances affect the win rate of a game of League of Legends in pro play, as of 2025.

The dataset contains **100,530 player rows** and 161 columns from 2025 LoL esports matches.

The dataset we are using consists of match-level and player-level data from 2025 LoL esports games. Each game has 12 rows (10 player rows + 2 team rows). Below are the key columns relevant to our analysis:

| Column | Description |
|--------|-------------|
| `gameid` | Unique identifier for each match |
| `result` | Win (1) or loss (0) for the player's team |
| `position` | Player role: top, mid, bot, jng, sup |
| `side` | Team side: 100 (Blue) or 200 (Red) |
| `datacompleteness` | "complete" = has early-game stats like goldat10 |
| `damageshare` | Player's share of team total damage |
| `golddiffat10` | Gold difference vs opponent at 10 min |
| `earnedgoldshare` | Share of team gold earned |
| `visionscore` | Vision control metric |
| `kills`, `deaths`, `assists` | Combat stats |

---

## Data Cleaning and Exploratory Data Analysis

For each League of Legends game there are 12 rows: 10 for each player and 2 for team summary. Because the team summary row is comprised of the game statistics of the players, we filtered the dataset to player rows only (removing the 2 team summary rows per game). We also converted binary columns (e.g. `firstblood`, `playoffs`) from 0/1 floats to booleans, and parsed the `date` column into a datetime type. These changes were made to make analysis easier and make the columns more readable.

Here is a sample of the cleaned DataFrame:

| gameid | result | position | side | damageshare | earnedgoldshare | visionscore | kills | deaths | assists |
|---|---|---|---|---|---|---|---|---|---|
| LOLTMNT03_179647 | 0 | top | Blue | 0.402 | 0.290 | 17 | 1 | 2 | 1 |
| LOLTMNT03_179647 | 0 | jng | Blue | 0.099 | 0.159 | 29 | 0 | 3 | 1 |
| LOLTMNT03_179647 | 0 | mid | Blue | 0.278 | 0.224 | 20 | 1 | 2 | 0 |
| LOLTMNT03_179647 | 0 | bot | Blue | 0.138 | 0.239 | 10 | 1 | 3 | 1 |
| LOLTMNT03_179647 | 0 | sup | Blue | 0.083 | 0.089 | 82 | 0 | 3 | 2 |

### Univariate Analysis

<iframe src="assets/damageshare_by_role.html" width="800" height="500" frameborder="0"></iframe>

The overall damage share distribution looks like an unusual multi-modal shape. Once broken down by role, the pattern makes sense — each role has its own distinct damage share distribution, kinda like how if we graphed shots on goal in soccer on all players, the defenders would be included with the attackers so the histogram would look like overlaid separate distributions. Support has overall lower damage share for example because their role in the game is provide buffs and protect the other players on the team. It is also interesting to note that the variance in each of the role's distributions with supports having the least (they roughly do around the same damage) while bot has the largest spread (some bot laners do way more than others) 

<iframe src="assets/visionscore_by_role.html" width="800" height="500" frameborder="0"></iframe>

Looks like supports are the primary vision getters — their vision score distribution is far higher than all other roles, reflecting their ward-focused playstyle. As stated above, supports do not contribute much damage, but they provide vision on the map to provide knowledge to the rest of the team. The distributions flipped between vision and damage between the highest and lowest distribution of damage/vision.

### Bivariate Analysis

<iframe src="assets/golddiff_by_result.html" width="800" height="500" frameborder="0"></iframe>

Gold is the in game currency that players recieve for obtaining kills/assists, objectives, and creep score (killing enemy minions, jungle monstrs, etc.). This currency is then used to buy in-game items that help the individual scale their player stats (stat increases include). Players on winning teams have a noticeably higher gold difference at 10 minutes on average, suggesting early gold leads correlate with winning the game.

<iframe src="assets/damageshare_by_position.html" width="800" height="500" frameborder="0"></iframe>

Damage share varies a lot by role — supports do less damage while carries (bot, mid) do more. Winners tend to have slightly higher damage shares than losers across most roles, but they generall have around the same mean.

### Interesting Aggregates

Average damage share, earned gold share, and vision score by role:

| position | mean_damageshare | earnedgoldshare | mean_visionscore |
|----------|-----------------|-----------------|-----------------|
| bot | 0.284 | 0.237 | 24.3 |
| jng | 0.165 | 0.193 | 28.7 |
| mid | 0.260 | 0.224 | 24.1 |
| sup | 0.086 | 0.091 | 67.4 |
| top | 0.205 | 0.255 | 20.8 |

Supports have drastically lower damage share and gold share but dominate vision score. Game stats vary by postion; What is considered "winning" in one role differs from the stats of winning in a different role. This role specialization is central to understanding why our baseline model struggled.

---

## Assessment of Missingness

### MNAR: `teamid`

The `teamid` column is **MNAR (Missing Not At Random)** because whether a team has an Oracle's Elixir-assigned ID depends on the team itself and its notability. Established organizations like G2 Esports, Weibo Gaming, and Team Solo Mid have IDs, while obscure amateur teams like *We Can Win* and *Black Dog* do not. We can pretty confidently say that any missing `teamid` value will not be within the existing `teamid`s in the column.

### Missingness Dependency: `golddiffat10`

We picked `golddiffat10` as our column to analyze, since around 8% of player rows are missing this value. We tested whether its missingness depends on `league` and `position`. `league` would make sense, since some leagues have full VODs due to being more popular, otherwise `golddiffat10` is not recorded. The `position` column shouldn't affect the missingness of `golddiffat10` since all 10 players in a game share the same data completeness. We used permutation tests with **TVD** as our test statistic since both `league` and `position` are categorical.

- **Null Hypothesis 1**: The distribution of league is the same whether golddiffat10 is missing or not.
- **Alternate Hypothesis 1**: The distribution of league differs between rows where golddiffat10 is missing vs. not missing.
- **Null Hypothesis 2**: The distribution of position is the same whether golddiffat10 is missing or not.
- **Alternate Hypothesis 2**: The distribution of position differs between rows where golddiffat10 is missing vs. not missing.
- **Significance level**: α = 0.05

The league permutation test yields a very small p-value of < 0.0001, so we **reject the null** — missingness of `golddiffat10` is **MAR on league** (some leagues consistently have incomplete early-game data).

The position permutation test yields a p-value of 1.0, so we **fail to reject the null** — missingness does **not depend on position**, which makes sense since all players in a game share the same data completeness.

<iframe src="assets/missingness_by_league.html" width="800" height="500" frameborder="0"></iframe>

Strangely, it looks like all the missing `golddiffat10` is all in LPL, China's league. While China is one of the strongest regions in LoL esports, it looks like China might be limiting its data from being easy-access.

---

## Hypothesis Testing

In a game of League of Legends, teams are places on two sides of the map: the red side which covers the bottom left side, and the blue side covers the top right side of the map. The game randomly decides which team is going to be on what side. Players are given a differing view of the map (upward or downward view) and their distances to objectives are different. Besides this, both sides are relatively same but are flipped and mirrored versions of each other.  However, we are curious if one side has a higher win rate than the other even though they are set up to be equal. We pose these set of hypotheses:


**Null hypothesis**: The blue side and red side have the same likelihood of winning in any pro LoL game.

**Alternative hypothesis**: The blue side is more likely to win than the red side in any pro LoL game.

**Test statistic**: Proportion of games the blue side wins.

**Significance level**: 0.05

After conducting a hypothesis test with 10,000 trials, we concluded that we can reject the null hypothesis with a p-value of < 0.0001. There seems to be an advantage of blue side in pro play.

---

## Framing a Prediction Problem

We frame this as a **binary classification** problem: given a player's in-game statistics, predict whether their team **won (1) or lost (0)** the match.

We chose `result` as our response variable because winning is the ultimate objective in League of Legends — understanding which player-level metrics best predict victory directly answers our central question.

We evaluate our model using **accuracy**, since the dataset is perfectly balanced (every game has exactly one winning team and one losing team, so wins and losses each make up 50% of player rows). A baseline of always predicting the majority class would give 50% accuracy, so anything above that is meaningful.

All features we use — such as `damageshare`, `golddiffat10`, `visionscore`, and `earnedgoldshare` — are statistics recorded during or at the end of a game, so they are available at the time of prediction (i.e., after the game concludes). We are **not** using any features that would only be known after the outcome is determined independently of the game itself.

---

## Baseline Model

The baseline model uses 4 features: `kill_participation` (created from kills/assists/teamkills), `damageshare`, `earnedgoldshare`, and `visionscore`, all run through a `StandardScaler` + `LogisticRegression` Pipeline. These are all quantitative features, so there's no categorical encoding needed here.

This baseline model has an accuracy of 0.5332, which is not much better than the constant model's accuracy of 0.5. This implies that these stats alone are definitely not great for predicting win/loss. Thinking about it a bit more, maybe these stats alone would be better for a different problem, predicting a player's role from stats. Although that model wouldn't be very useful.

---

## Final Model

**Model choice**: Random Forest outperformed both Logistic Regression and an untuned Decision Tree on the final feature set, so it was selected as the final model.

**Hyperparameters tuned** via `GridSearchCV` (5-fold CV):
- `n_estimators`: number of trees in the forest
- `max_depth`: maximum depth of each tree (controls overfitting)

**New engineered features**:
- `damage_efficiency`: `damagetochampions / (deaths + 1)`, measures how much damage a player deals per death (adding 1 to avoid division by zero). A high value means the player is dealing a lot of damage while dying infrequently — buying the right items and playing efficiently.
- `carry_impact`: a weighted score (kill participation 35%, damage share 30%, earned gold share 20%, damage efficiency 10%, vision score 5%) to quantify a player's overall impact. Higher weights on kill participation and damage share, since kills matter the most in LoL, kinda like home runs vs almost anything else for a baseball player.

These two features add non-linear relationships that raw stats alone do not capture, which is why the Random Forest achieves higher accuracy than the baseline Logistic Regression model.

---

## Fairness Analysis

Since a lot of our player performance metrics are based on damage and kills, a valid concern is that this model is biased against support players, whose primary role isn't to deal damage and/or participate in kills. The model might have learned that low damage/kill stats correlate with losing, and will systematically predict supports as losers even when their team wins.

**Groups:**
- **Group X**: Players which have Support as their position
- **Group Y**: All other positions (Top, Middle, Bottom, Jungle)

**Evaluation metric**: False Negative Rate, the proportion of actual wins that the model incorrectly predicts as losses

**Null hypothesis**: Our model is fair. The False Negative Rate for supports and non-supports is the same.

**Alternative hypothesis**: Our model is unfair. The False Negative Rate is higher for supports than non-supports.

**Test statistic**: FNR(supports) − FNR(non-supports)

**Significance level**: 0.05

Based on the p-value of 0.003 we reject the null and conclude the model has a higher false negative rate for supports, unfairly underpredicting wins for support players.
