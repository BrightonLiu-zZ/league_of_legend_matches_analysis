## Step 1

### Goal
I study: *When it's behind, which position hurts its team’s chance to win the least?*  
In other words, in situations where the game is going poorly, and if a player is trailing their lane opponent in gold, which role (top, jng, mid, bot, sup) is the most “forgiving” for team win rate.

### Data
Our raw dataset is on all of the professional League of Legends games that have taken place in 2022.  
Each game contributes one row per player and 2 rows for team statistics, which result in **150,588 rows** in the raw dataset, with **>100 columns** of in-game stats. The dataset contains 12 rows per game, one row per player and 2 rows of team statistics. Furthermore, there are over **164 columns** of nearly all kinds of data you could collect on a League of Legends match.

#### Columns used
- `gameid`: unique match ID.  
- `league`: tournament/league label.  
- `side`: team side (“blue” or “red”).  
- `position`: role (“top”, “jng”, “mid”, “bot”, “sup”).  
- `result`: team result for that player’s team (1 = win, 0 = loss).  
- `golddiffat15`: amount of gold ahead of the opponent of corresponding position.  
- `deficit15` (boolean): true when the team’s `golddiffat15` is negative (team is behind).  
- `comeback15` (boolean): true if `deficit15` == True and the team still won.  
- `has_more_kills` (boolean): true when a player has more kills than their lane opponent.

### Why it matters
Players, coaches, and analysts can use this to understand where deficits are least costly — i.e., which role can *“be behind and still not sink the game”* — to inform draft, lane assignments, and mid-game decision making.


## Step 2

### Data Cleaning
**For the data cleaning part, I have done these steps:**

1. I removed rows that contained team-level statistics rather than player-level information, since our analysis focuses on individual positions.  

2. I narrowed down the dataset to only the columns that are directly relevant to our question (`gameid`, `league`, `side`, `position`, `has_more_kills`, `result`, `golddiffat15`, `deficit15`, `comeback15`).  

3. Since we need data when the game time is at 15 minutes, I filtered out rows with `gamelength < 900s`.  

---

### Univariate Analysis

#### Counts by Position (Bar Chart)

<iframe src="assets/images/position_counts.html" width="100%" height="520"></iframe>

The distribution is almost uniform (top/mid/bot/jng ≈ 19.7 – 19.9%, sup ≈ 20.8% which is slightly higher).  
This indicates that the sample sizes for each position are relatively balanced, so the future data analyzing result will not be unstable due to a small sample size for a certain position. However, it should also be noted that the sample size for **sup** is slightly larger, so its standard error is slightly smaller.

---

#### The ECDF of 15-minute Gold Diff

<iframe src="assets/images/ecdf_gold15.html" width="100%" height="520"></iframe>

The curve is **S-shaped**, with the main body concentrated in the range of -2500 to -500, and there is a long tail on the left.  
This means that “lightly/moderately behind” is more common, and samples of “severely behind” are rare. If we do not classify them, and a certain position appears more frequently in the “lightly behind” games, it may lead to an overestimation.  

*Note:* Since ECDF is conditional on teams that were already behind at 15 minutes, the graph is not centered on Gold diff at 15 = 0.  

---

### Bivariate Analysis

#### Comeback Rate vs Position (Bar Chart)

<iframe src="assets/images/position_comeback_rate.html" width="100%" height="520"></iframe>

Under the condition of “both gold and kills are behind the opponents,” **SUP ≈ 21%** is the highest, **JNG ≈ 15%** is second, **TOP/MID** are in the middle (≈ 12 – 14%), and **BOT ≈ 9%** is the lowest.

---

#### 15-minute Gold Deficit by Position (Box Plot)

<iframe src="assets/images/position_gold_diff_box.html" width="100%" height="520"></iframe>



The distribution of `golddiffat15` at each position is generally similar, with the median around -500 gold, and there is a relatively deep left tail (extremely behind). From a visual perspective, it doesn’t seem like “a certain position has a systematically lighter or severer behind.”  

This mitigates the “confounding” concern, as the high comeback rate of **SUP** is not significantly due to encountering a lighter disadvantage.  

### Interesting Aggregates

<iframe src="assets/images/pivot_rate_pct.html" width="100%" height="520"></iframe>

#### Pattern
The lighter the behind, the higher the comeback rate (for all positions from severe to slight, the rate is increasing, except for top from mild → slight).

---

#### Position ranking (consistent except for the most severe cases)
Among the **moderate / mild / slight** categories:  
- **SUP** has the highest score (16.4 / 16.8 / 21.7).  
- **JNG** follows with the second-highest score (12.7 / 16.5 / 16.7).  
- **BOT** has the lowest score (9.4 / 10.4 / 10.1).  

**Severe (< -2500)** layer presents an exception: the **TOP’s 14.3%** is even higher than the SUP’s 8.3%.

---

#### Insight
Under the condition that *“when the team is behind and the position’s kill is not higher than the opponent’s corresponding position,”*  
- The **support** is the most capable of “saving the day” (three layers in agreement).  
- The **jungler** generally ranks second.  
- The **bottom lane (BOT)** is the weakest.  
- Only when severely behind, the performance of **TOP** is exceptionally high.
