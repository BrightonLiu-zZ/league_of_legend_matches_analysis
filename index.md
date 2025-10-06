---
title: LoL Comeback Analysis (DSC80)
layout: default
---

<!-- ====== Minimal inline styles for a crisp, modern look ====== -->
<style>
:root { --brand:#0b66ff; --ink:#0b0e16; --muted:#586069; --card:#ffffff; --line:#e5e7eb;}
* { box-sizing: border-box; }
.hero { text-align:center; padding: 1.25rem 0 .5rem; }
.hero h1 { margin:.25rem 0 .25rem; font-size: clamp(1.6rem, 3.5vw, 2.4rem); line-height:1.15; }
.hero p.lead { margin:.4rem auto 0; color: var(--muted); max-width: 60ch; }
.badges { display:flex; flex-wrap:wrap; gap:.5rem; justify-content:center; margin: .9rem 0 0; }
.badge { font-size:.9rem; border:1px solid var(--line); padding:.35rem .6rem; border-radius:999px; background:rgba(0,0,0,.03); }
.quicklinks { display:flex; gap:.5rem; flex-wrap:wrap; justify-content:center; margin: 1rem 0 0; }
.quicklinks a { text-decoration:none; padding:.55rem .8rem; border-radius:10px; border:1px solid var(--line); background:var(--card); }
.quicklinks a.primary { border-color: transparent; background: var(--brand); color: white; }

.section { margin: 2rem 0; }
.section h2 { margin: 0 0 .4rem; font-size: clamp(1.2rem, 2.5vw, 1.6rem); }
.subtle { color: var(--muted); }

.callout { border-left:6px solid var(--brand); background:rgba(11,102,255,.06); padding:1rem; border-radius:10px; }
.card-grid { display:grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); gap: 1rem; }
.card { border:1px solid var(--line); background:var(--card); border-radius:12px; padding:1rem; }
.card h3 { margin:.2rem 0 .4rem; font-size:1.05rem; }
.card p { margin:.3rem 0 .6rem; color: var(--muted); }

/* ------- Fix for cropped/scrolling interactive figures ------- */
.card-grid.keyfig { grid-template-columns: 1fr; }      /* single column so Plotly fits */
.fig-embed {
  display:block;
  width:100%;
  height: 520px;                 /* fits typical 700x450 Plotly exports */
  border:0;
  background:#fff;
  border-radius:10px;
  outline: 1px solid var(--line);
}
/* On very wide screens, allow two columns while keeping enough width */
@media (min-width: 1500px) {
  .card-grid.keyfig { grid-template-columns: repeat(2, minmax(680px, 1fr)); }
}

details { border:1px dashed var(--line); border-radius:10px; padding:.8rem; }
details summary { cursor:pointer; font-weight:600; }

.footer-note { margin-top: 2rem; font-size:.95rem; color: var(--muted); }
</style>

<div class="hero">
  <h1>When behind at 15:00, which position is most <em>forgiving</em>?</h1>
  <p class="lead">
    Using all professional League of Legends matches in 2022 (150,588 player rows, 164 features),
    we study which role hurts team win chance the least when trailing at 15 minutes.
  </p>
  <div class="badges">
    <span class="badge">📅 Season: 2022</span>
    <span class="badge">📊 150,588 rows</span>
    <span class="badge">🧭 Snapshot: 15:00</span>
    <span class="badge">🎯 Final metric: ROC&nbsp;AUC</span>
  </div>
  <div class="quicklinks">
    <a class="primary" href="assets/images/position_comeback_rate.html">Top finding (interactive)</a>
    <a href="assets/images/pivot_rate_pct.html">Deficit × Role (interactive)</a>
    <a href="assets/images/confusion_matrix.html">Confusion matrix</a>
    <a href="https://github.com/brightonliu-zz/league_of_legend_matches_analysis">GitHub repo</a>
  </div>
</div>

---

<div class="section">
<h2>TL;DR</h2>
<div class="callout">
<p><strong>Support (SUP) sustains the highest comeback rate when a team is behind at 15:00</strong> and the lane is not ahead in kills, consistently across mild → moderate deficits. 
Jungle (JNG) is typically second; Bottom (BOT) is most fragile. In severe deficits, Top shows an exception with relatively stronger resilience.</p>
<p>Our <strong>final model is a simple logistic regression</strong> using 15-minute gold and CS differences (ROC&nbsp;AUC ≈ <strong>0.73</strong> test). It outperformed more complex baselines here and remained interpretable.</p>
</div>
</div>

<div class="section">
<h2>Key Figures (interactive)</h2>
<div class="card-grid keyfig">

  <div class="card">
    <h3>Comeback rate by role</h3>
    <p class="subtle">SUP ≈ highest; JNG ≈ second; BOT ≈ lowest when behind at 15:00.</p>
    <iframe class="fig-embed" src="assets/images/position_comeback_rate.html" loading="lazy"></iframe>
  </div>

  <div class="card">
    <h3>Deficit severity × role</h3>
    <p class="subtle">Comeback rate rises as deficits get lighter; Top stands out only in severe cases.</p>
    <iframe class="fig-embed" src="assets/images/pivot_rate_pct.html" loading="lazy"></iframe>
  </div>

  <div class="card">
    <h3>Final model performance</h3>
    <p class="subtle">Baseline logistic regression (gold/CS at 15:00) — ROC&nbsp;AUC ≈ 0.73 test.</p>
    <iframe class="fig-embed" src="assets/images/confusion_matrix.html" loading="lazy"></iframe>
  </div>

</div>

<details>
  <summary>More visuals (open)</summary>
  <ul>
    <li><a href="assets/images/position_counts.html">Counts by Position</a></li>
    <li><a href="assets/images/ecdf_gold15.html">ECDF of 15′ Gold Difference (teams already behind)</a></li>
    <li><a href="assets/images/position_gold_diff_box.html">15′ Gold Deficit by Position (box)</a></li>
    <li><a href="assets/images/tvd_perm_hist_test_1.html">Missingness test — by league (TVD permutation)</a></li>
    <li><a href="assets/images/missing_rate_by_league.html">Missing rate by league</a></li>
    <li><a href="assets/images/miss_rate_by_side.html">Missing rate by side</a></li>
    <li><a href="assets/images/tvd_perm_hist.html">Missingness test — by side (TVD permutation)</a></li>
    <li><a href="assets/images/fairness_perm_side_auc.html">Fairness: AUC Blue vs Red (permutation)</a></li>
  </ul>
</details>
</div>

<div class="section">
<h2>Data & Cleaning (brief)</h2>
<ul>
  <li><strong>Scope:</strong> All professional matches in <strong>2022</strong>. One row per player and two rows for team stats.</li>
  <li><strong>Size:</strong> <strong>150,588</strong> rows, <strong>164</strong> columns. Focused columns: <code>gameid</code>, <code>league</code>, <code>side</code>, <code>position</code>, <code>result</code>, <code>golddiffat15</code>, <code>deficit15</code>, <code>comeback15</code>, <code>has_more_kills</code>.</li>
  <li><strong>Filters:</strong> Kept matches with <code>gamelength ≥ 900s</code>; focused on player-level rows for position analysis; dropped leagues where 15′ timeline is systematically unavailable (MAR).</li>
</ul>
</div>

<div class="section">
<h2>Hypothesis Test (role resilience)</h2>
<p><strong>H₀:</strong> Within each 15′ deficit bin, SUP and non-SUP have the same comeback rate.  
<strong>H₁:</strong> SUP has a higher comeback rate.</p>
<p><em>Method:</em> Stratified (by deficit bins) permutation test on win rate differences; weighted by bin sizes.</p>
<p><strong>Result:</strong> p-value ≈ <strong>0.001</strong> (B=1000) → reject H₀. Supports “SUP is most forgiving when behind.”</p>
</div>

<div class="section">
<h2>Prediction Task (15:00 → final result)</h2>
<ul>
  <li><strong>Unit:</strong> Team-game; prediction made at <strong>15:00</strong>.</li>
  <li><strong>Target:</strong> <code>result</code> (win=1).</li>
  <li><strong>Features:</strong> <code>golddiffat15</code>, <code>csdiffat15</code>.</li>
  <li><strong>Model:</strong> Logistic regression (baseline &amp; final).  
  <br><em>Why it won:</em> Best ROC AUC on test and remains interpretable.</li>
</ul>
</div>

<div class="section">
<h2>Fairness Check (Blue vs Red)</h2>
<p><em>Metric:</em> ROC AUC by side; <em>Test:</em> label permutation on group tags.</p>
<p><strong>Outcome:</strong> AUC(Blue)=0.626, AUC(Red)=0.628, Δ=−0.0014; two-sided p=0.933 → no evidence of side bias.</p>
<p><a href="assets/images/fairness_perm_side_auc.html">See the permutation distribution ⟶</a></p>
</div>

<hr>