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

<iframe src="assets/images/position_counts.html" width="80%" height="500"></iframe>

The distribution is almost uniform (top/mid/bot/jng ≈ 19.7 – 19.9%, sup ≈ 20.8% which is slightly higher).  
This indicates that the sample sizes for each position are relatively balanced, so the future data analyzing result will not be unstable due to a small sample size for a certain position. However, it should also be noted that the sample size for **sup** is slightly larger, so its standard error is slightly smaller.

---

#### The ECDF of 15-minute Gold Diff

<iframe src="assets/images/ecdf_gold15.html" width="80%" height="500"></iframe>

The curve is **S-shaped**, with the main body concentrated in the range of -2500 to -500, and there is a long tail on the left.  
This means that “lightly/moderately behind” is more common, and samples of “severely behind” are rare. If we do not classify them, and a certain position appears more frequently in the “lightly behind” games, it may lead to an overestimation.  

*Note:* Since ECDF is conditional on teams that were already behind at 15 minutes, the graph is not centered on Gold diff at 15 = 0.  

---

### Bivariate Analysis

#### Comeback Rate vs Position (Bar Chart)

<iframe src="assets/images/position_comeback_rate.html" width="80%" height="500"></iframe>

Under the condition of “both gold and kills are behind the opponents,” **SUP ≈ 21%** is the highest, **JNG ≈ 15%** is second, **TOP/MID** are in the middle (≈ 12 – 14%), and **BOT ≈ 9%** is the lowest.

---

#### 15-minute Gold Deficit by Position (Box Plot)

<iframe src="assets/images/position_gold_diff_box.html" width="80%" height="500"></iframe>



The distribution of `golddiffat15` at each position is generally similar, with the median around -500 gold, and there is a relatively deep left tail (extremely behind). From a visual perspective, it doesn’t seem like “a certain position has a systematically lighter or severer behind.”  

This mitigates the “confounding” concern, as the high comeback rate of **SUP** is not significantly due to encountering a lighter disadvantage.  

### Interesting Aggregates

<iframe src="assets/images/pivot_rate_pct.html" width="80%" height="500"></iframe>

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

## Step 3
## Assessment of Missingness

### What we test
We are concerned about whether the absence of the 15-minute money difference (`golddiffat15`) is dependent on certain observed variables.  
Let \( Y = 1\{\text{`golddiffat15` is NA}\} \).

The test statistic used is **TVD (Total Variation Distance)**, which compares the distribution differences of “missing group” vs “non-missing group” on a categorical variable:

\[
\text{TVD} = \tfrac{1}{2} \sum_i \, \lvert p_i - q_i \rvert
\]

---

### Test 1
**\(H_0\):** The distribution of *league* is the same in the missing `golddiffat15` data group and the non-missing `golddiffat15` data group.  
**\(H_A\):** The distribution of *league* is different between the two groups.

<iframe src="assets/images/tvd_perm_hist_test_1.html" width="80%" height="500"></iframe>

**Test result:**  
Observed TVD = 0.9926, p-value ≈ 0.002 \((B = 500)\).

The missing rate by league group is shown as follows: the missing rates for ASCI, DCup, LDL, and LPL are 1.0000, WLDs ≈ 0.0903, while most other leagues are 0.

**Interpretation:**  
The lack of high values is concentrated in a few leagues, indicating that the absence of `golddiffat15` is significantly dependent on *league*.  
This is consistent with the data generation process (some leagues’ timelines are missing).  
Therefore, the missing mechanism in `golddiffat15` is **MAR** rather than **NMAR**.

---

### Test 2
**\(H_0\):** The distribution of *side* is the same in the missing `golddiffat15` group and the non-missing group.  
**\(H_A\):** The distribution of *side* is not the same in the missing group and the non-missing group.

<iframe src="assets/images/miss_rate_by_side.html" width="80%" height="500"></iframe>

---

<iframe src="assets/images/tvd_perm_hist.html" width="80%" height="500"></iframe>

**Test result:**  
Observed TVD = 0.0000, p-value = 1.000 \((B = 500)\).  
The missing rates of Blue and Red are both 0.1508.

**Interpretation:**  
The absence has nothing to do with the team side.


## Step 4
### Hypothesis Testing

#### Question I want to investigate
In the game where *“the team is behind (`deficit15 == True`) and the position’s kill is less than the opponent (`has_more_kills == False`),* is the win rate of the **support (SUP)** role higher than that of other positions?

---

### Groups
We compare **SUP vs non-SUP**, stratified by 15-minute gold-deficit bins:

- Severe: \((-\infty, -2500)\)  
- Moderate: \([-2500, -1500)\)  
- Mild: \([-1500, -500)\)  
- Slight: \([-500, 0)\)

---

#### Hypotheses (one-sided)
- **\(H_0\):** Within each deficit bin, SUP and non-SUP have the **same** comeback rate.  
- **\(H_A\):** Within each deficit bin, the SUP comeback rate is **higher**.

---

<iframe src="assets/images/tvd_perm_hist_test_1.html" width="80%" height="500"></iframe>

#### Test Statistic
Weighted stratified difference in win rates:

\[
T = \sum_b w_b \big(\hat{p}_{\text{SUP},b} - \hat{p}_{\text{nonSUP},b}\big), 
\qquad 
w_b = \frac{n_b}{\sum_b n_b}
\]

---

#### Result
The permutation test yielded a p-value of **0.001** (with \(B = 1000\)), which is smaller than any common significance levels (like 0.05).

**Decision:** Since the p-value is smaller than 0.05, we reject the null.

---

### Conclusion
The data provides strong evidence to reject \(H_0\).  
In other words, in matches where the team was behind and the position’s kills were lower, the **comeback rate of the support role** was significantly higher than that of the non-support roles.

## Step 5
### Framing a Prediction Problem

#### Goal (what to predict)
I frame this as a **binary classification task**: given information available by the 15-minute mark of a match, predict whether that team will win the game  
(\(`result` = 1 if the team eventually wins, else 0\)).

---

#### Unit of prediction
Each row is a **team–game** (e.g., one row for Blue side and one for Red side in the same `gameid`).  
When splitting data, I keep both sides from the same game in the same split.

---

#### When the prediction is made
We predict at **15:00 in-game time**, treating this as a frozen snapshot.  
So all features must be knowable at 15:00 to predict what happens at the end of the game.

---

#### Response variable
- **`result`**: whether the team wins the match (categorical).

---

#### Predictive features
1. **`golddiffat15`** — gold difference at 15 minutes (numerical).  
2. **`csat15`** — creep score (number of farmable units a player last-hit) at 15 minutes (numerical).  
3. **`side`** — Blue/Red team (categorical, known before the game starts).

---

#### Why this prediction is useful
This task estimates win probability from a mid-game snapshot, which helps us quantify:
- How much early advantages translate into final outcomes.  
- How often teams come back from early deficits.  

---

#### Evaluation Metric
I use **ROC AUC** as the main evaluation metric, because our models output probabilities, and we care about correctly ranking teams by win likelihood.  
ROC AUC is threshold-independent and works well even with class imbalance.

I do **not** use accuracy as the primary metric, since it depends on an arbitrary cutoff and can be misleading.  
For interpretability, I also report accuracy with confusion matrices, but model selection is based on ROC AUC.

## Step 6

### Baseline Model

#### Model
I built a simple **logistic regression** as the baseline.  
The model uses only two features that are available by the 15-minute mark:  

- `golddiffat15` (team gold difference at 15 minutes)  
- `csdiffat15` (average lane CS difference at 15 minutes)  

Both are quantitative features, so no encoding was needed.  
We did not include `side` to avoid leakage and to leave it for fairness analysis later.

The model is defined as:

\[
P(\text{win}) = \frac{1}{1 + \exp \big( - (b_0 + b_1 \cdot \text{golddiffat15} + b_2 \cdot \text{csdiffat15}) \big)}
\]

- \(b_0\) is the intercept (bias).  
- \(b_1\) is the coefficient for gold difference at 15 minutes.  
- \(b_2\) is the coefficient for CS difference at 15 minutes.  

---

#### Evaluation
The model was evaluated with **ROC AUC** as the primary metric.  
I used 5-fold cross-validation on the training set and a separate test set to check generalization.

---

#### Performance
- **ROC AUC (cross-validation):** 0.742 ± 0.012  
- **ROC AUC (test):** 0.732  

---

#### Interpretation
A ROC AUC around **0.73** means if we randomly pick a winning team and a losing team, the baseline model has a **73% chance** of ranking the winner’s probability higher than the loser’s.

---

#### What the model would look like (coefficients I got)
\[
P(\text{win}) = \frac{1}{1 + \exp \big( -(-0.000003 + 0.000255 \cdot \text{golddiffat15} + 0.101135 \cdot \text{csdiffat15}) \big)}
\]

## Step 7
### Final Model

#### What we tried to add (and why)
- **Severity bins for 15-minute deficits (gold and CS):** We expected non-linear threshold effects (being slightly behind vs. severely behind may have very different impacts).  
- **Interaction term gold × CS:** We expected “both behind in gold and CS” to hurt more than the sum of each deficit.  

These additions follow the game process, as early deficits come from farm and lane pressure, and combined disadvantages should matter more.

---

#### Models compared and how we tuned them
1. **Baseline logistic regression**  
   - Two quantitative features: `golddiffat15`, `csdiffat15`.  
   - No scaling, no tuning.  

2. **Logistic regression with feature engineering**  
   - Adds bins and interaction.  
   - Pipeline includes one-hot for bins and standardization for numeric features.  
   - Hyperparameters tuned with GridSearchCV.  

3. **Random Forest**  
   - Non-linear model.  
   - Tuned with GridSearchCV on depth, trees, and split/leaf sizes.  

---

#### Evaluation Protocol
- **Primary metric:** ROC AUC.  
- **Model selection:** 5-fold StratifiedGroupKFold CV on the training set.  
- Hyperparameters tuned only on the training set, final test ROC AUC reported once.  

---

#### Results (ROC AUC)
- Logistic (baseline): CV ≈ 0.742 ± 0.012; Test ≈ 0.732.  
- Logistic + feature engineering: CV ≈ 0.7419 ± 0.0117; Test ≈ 0.732 (slight changes but not meaningful).  
- Random Forest: CV ≈ 0.662 ± 0.006; Test ≈ 0.656 (below baseline).  

---

#### Conclusion (why the baseline is our final model)
The **baseline logistic regression** has the highest ROC AUC on the test set and generalizes well (test score close to CV).  
The added bins and interaction did not help, likely because the two original 15-minute features already capture most of the predictive signal, and the extra terms are either redundant or regularized away.  
Random Forest underperformed with this small feature set.  

Therefore, we choose the **baseline logistic regression** as the final model for its **best ranking performance, simplicity, and interpretability**.  

---

#### Final model formula (for the website)
\[
P(\text{win}) = \frac{1}{1 + \exp \Big( - \big( -0.000003 + 0.000255 \cdot \text{golddiffat15} + 0.101135 \cdot \text{csdiffat15} \big) \Big) }
\]


<iframe src="assets/images/confusion_matrix.html" width="80%" height="500"></iframe>
I want to end the whole model selection section with this confusion matrix for the baseline model to clarify the actual performance of our final model:  Although its overall ROC AUC score is decent, the model still misclassify fair amount of games, so there is still room for improvement, like by adding more useful features to it. 

## Step 8
### Fairness Analysis

#### Question
Here I am trying to answer this question:  
*“Does my model perform worse for individuals in Group X than it does for individuals in Group Y?”*

---

#### Groups
- **Group X:** Blue side  
- **Group Y:** Red side  

We test whether our final model treats the two sides differently.

---

#### Evaluation metric
- **ROC AUC** (threshold-independent; measures ranking quality).  

---

#### Test Statistic
Difference in AUC:

\[
T = \text{AUC}(\text{Blue}) - \text{AUC}(\text{Red})
\]

---

#### Hypotheses
- **\(H_0\):** \(\text{AUC}(\text{Blue}) = \text{AUC}(\text{Red})\).  
  Any observed difference is due to random variation.  
- **\(H_A\):** \(\text{AUC}(\text{Blue}) \neq \text{AUC}(\text{Red})\).  
  (We report a two-sided test; a one-sided “Blue worse” version is also shown.)

---

#### Method
We perform a **permutation test** on the test set:  

- Keep true labels and predicted probabilities fixed.  
- Randomly shuffle the Blue/Red group labels many times (\(B = 2000\)).  
- Recompute the AUC difference each time.  
- Compare the observed difference against this null distribution.  

---

#### Results
- \(\text{AUC}(\text{Blue}) = 0.626\), \(\text{AUC}(\text{Red}) = 0.628\)  
  → Observed difference = \(-0.0014\).  

- Two-sided \(p\)-value = 0.9330.  

- One-sided \(p\)-value for “Blue worse” = 0.4738.  

<iframe src="assets/images/fairness_perm_side_auc.html" width="80%" height="500"></iframe>

---

#### Conclusion
We do not find evidence that the model’s ranking performance differs by side.  
The observed AUC gap is tiny and consistent with random variation at \(\alpha = 0.05\).
