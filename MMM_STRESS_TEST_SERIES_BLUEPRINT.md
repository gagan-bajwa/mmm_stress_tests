# MMM Stress Test Series
## A reproducible research programme for testing when Media Mix Models recover the truth — and when they do not

**Status:** Research blueprint  
**Primary framework:** Google Meridian  
**Secondary sanity checks:** OLS / Ridge / simple Bayesian regression where useful  
**Primary KPI:** Revenue  
**Primary evaluation target:** Recovery of known causal incremental revenue / ROAS  
**Recommended cadence:** One core experiment every 1–2 weeks  
**Intended outputs:** GitHub repository + LinkedIn posts + short videos + occasional long-form articles/newsletters

---

# 1. Why this series exists

Most MMM case studies use real-world marketing data.

That creates a fundamental problem:

> We usually do not know the true incremental effect of each media channel.

If an MMM says:

- Meta ROAS = 3.2
- Search ROAS = 5.1
- TV ROAS = 1.4

we can assess whether the result looks plausible, whether the model predicts well, whether MCMC converged, whether the response curves look reasonable, and whether experiments support the estimates.

But we still do not know the full ground truth.

The purpose of this project is to create synthetic marketing datasets where the true data-generating process (DGP) is known exactly.

We then deliberately introduce common MMM problems one at a time:

- weak variation
- always-on channels
- multicollinearity
- priors that disagree with the truth
- endogeneity
- omitted variables
- low media contribution
- noisy outcomes
- short time series
- baseline over-flexibility
- incorrect adstock assumptions
- incorrect saturation assumptions
- poor holdout behaviour
- experiment calibration mismatch
- multiple problems simultaneously

The central research question is:

> **Under what conditions can an MMM recover the causal marketing effect that actually generated the data?**

The project is not intended to "prove MMM does not work".

The more useful objective is:

> **Map the conditions under which MMM estimates are reliable, weakly identified, prior-sensitive, unstable, or misleading.**

---

# 2. The central philosophy

Every experiment should follow one rule:

> **Change one important thing at a time.**

Do not begin with:

- high correlation,
- endogeneity,
- always-on media,
- omitted confounders,
- incorrect priors,
- strong saturation,
- and short data

all in the same simulation.

If the model fails, you will not know why.

Instead use a controlled research structure:

1. Define a clean DGP.
2. Confirm the model can recover truth.
3. Change one condition.
4. Repeat the simulation many times.
5. Compare recovery to ground truth.
6. Only later combine failure modes.

The series should therefore progress from **clean identification** to **realistic messiness**.

---

# 3. What should count as "truth"?

For this series, the most useful ground-truth quantities are:

## 3.1 Incremental revenue

For channel \(m\):

\[
IncrementalRevenue_m
=
Y(\text{historical media})
-
Y(\text{channel m set to zero})
\]

Because we control the synthetic DGP, we can calculate this exactly.

---

## 3.2 Average ROAS

\[
ROAS_m
=
\frac{IncrementalRevenue_m}{Spend_m}
\]

This should be the main quantity reported publicly because marketers understand it.

---

## 3.3 Marginal ROAS

When saturation exists:

\[
mROAS_m(s)
=
\frac{d IncrementalRevenue_m(s)}{d Spend_m}
\]

Average ROAS and marginal ROAS are no longer equivalent once the response curve is nonlinear.

---

## 3.4 Contribution

\[
Contribution_m
=
\frac{IncrementalRevenue_m}{TotalRevenue}
\]

This is particularly useful when media represents a small part of total sales.

---

## 3.5 Raw coefficients

Use coefficients only when the DGP is actually linear.

Once adstock and Hill saturation are introduced, a raw regression coefficient is generally not an intuitive business quantity.

For LinkedIn/video communication, prioritise:

1. true ROAS
2. estimated ROAS
3. true contribution
4. estimated contribution
5. total-media incremental revenue
6. uncertainty

---

# 4. The common synthetic marketing world

Use one master synthetic system across the series wherever possible.

That gives continuity.

Recommended channels:

| Channel | Role | Base true ROAS | Typical behaviour |
|---|---|---:|---|
| Meta | Paid social | 2.0 | Variable / upper-mid funnel |
| Search | Paid search | 5.0 | Often demand-correlated |
| TV | Offline brand | 1.5 | Bursty |
| Display | Display/programmatic | 1.2 | Optional fourth channel |

For the first few experiments, three channels are enough.

TV acts as an internal control when Meta and Search are deliberately made correlated.

---

# 5. Common time structure

Start with:

```python
N_WEEKS = 156
```

Three years is a useful default.

Also test:

- 52 weeks
- 78 weeks
- 104 weeks
- 156 weeks
- 208 weeks

in the dedicated sample-size stress test.

Use weekly data because this resembles common national MMM practice.

---

# 6. Baseline data-generating process

A general baseline can be:

\[
Baseline_t =
\alpha
+ Trend_t
+ Seasonality_t
+ Promotion_t
+ Price_t
+ Macro_t
\]

Example:

\[
Baseline_t =
2,000,000
+ 4,000t
+ 500,000 \sin(2\pi t / 52)
+ 150,000 \cos(2\pi t / 52)
+ 400,000 Promo_t
- 250,000 PriceIndex_t
+ 200,000 Macro_t
\]

Keep every component stored separately.

Do not only save the final KPI.

Your synthetic dataframe should ideally contain:

```text
week
revenue
true_baseline
trend
seasonality
promo
price_index
macro
meta_spend
search_spend
tv_spend
true_meta_incremental
true_search_incremental
true_tv_incremental
true_total_media_incremental
```

This makes debugging much easier.

---

# 7. Media transformations

## Phase 1: linear media

For the first identification experiments:

\[
MediaEffect_{m,t} = ROAS_m \times Spend_{m,t}
\]

This is deliberately simple.

It isolates identification.

---

## Phase 2: geometric adstock

A standard recursive form:

\[
A_t = Spend_t + \alpha A_{t-1}
\]

where:

\[
0 \le \alpha < 1
\]

Example true decay parameters:

| Channel | True adstock alpha |
|---|---:|
| Meta | 0.30 |
| Search | 0.10 |
| TV | 0.70 |

TV should have a longer carryover than Search.

---

## Phase 3: Hill saturation

One useful Hill form:

\[
Hill(x)
=
\frac{x^s}{x^s + ec^s}
\]

where:

- \(ec\) controls the half-saturation point
- \(s\) controls slope/shape

Example values:

| Channel | ec | slope |
|---|---:|---:|
| Meta | 60,000 | 1.5 |
| Search | 45,000 | 1.3 |
| TV | 150,000 | 1.8 |

Scale carefully so incremental revenue has realistic magnitude.

---

# 8. Final synthetic outcome

General form:

\[
Revenue_t =
Baseline_t
+
\sum_m MediaEffect_{m,t}
+
\epsilon_t
\]

with:

\[
\epsilon_t \sim Normal(0,\sigma)
\]

Later experiments may deliberately violate this structure.

---

# 9. Monte Carlo design

Never rely on one simulated dataset.

A single dataset can produce a surprising estimate because of random noise.

For every condition:

```python
N_SIMS = 100
```

Minimum.

For cheap models:

```python
N_SIMS = 500
```

or even:

```python
N_SIMS = 1000
```

For Meridian, computational cost may require:

```python
N_SIMS = 20–100
```

depending on GPU resources and posterior sampling settings.

A useful strategy is:

1. Explore the full simulation grid using OLS/Ridge/simple Bayesian models.
2. Select representative conditions.
3. Run those conditions through Meridian.
4. Confirm that the qualitative conclusion remains.

---

# 10. Random seeds

Every run must be reproducible.

Example:

```python
MASTER_SEED = 20260831

seed = MASTER_SEED + scenario_id * 100_000 + simulation_id
rng = np.random.default_rng(seed)
```

Never rely on global random state.

Save the seed with every result.

---

# 11. The standard MMM Stress Test scorecard

Every experiment should output the same core metrics.

This is essential.

## 11.1 Channel ROAS bias

\[
Bias_m
=
E[\hat{ROAS}_m] - ROAS_m
\]

---

## 11.2 Relative error

\[
RelativeError_m
=
\frac{|\hat{ROAS}_m - ROAS_m|}{ROAS_m}
\]

Useful thresholds:

- > 25% wrong
- > 50% wrong
- > 100% wrong

---

## 11.3 RMSE

\[
RMSE_m
=
\sqrt{
\frac{1}{N}
\sum_i
(\hat{ROAS}_{mi}-ROAS_m)^2
}
\]

---

## 11.4 Credible interval coverage

If you report a 90% credible interval:

\[
Coverage =
P(TrueROAS \in 90\%CI)
\]

Across repeated simulations, ideal calibration would produce approximately the advertised coverage.

This is one of the strongest diagnostics in the entire project.

---

## 11.5 Interval width

A model can technically cover the truth by producing uselessly wide intervals.

Therefore track:

\[
Width = UpperCI-LowerCI
\]

---

## 11.6 Wrong-sign probability

For positive causal effects:

\[
P(\hat{ROAS}<0)
\]

If the model constrains ROI positive through the prior, track a different failure indicator such as near-zero or implausibly high ROI.

---

## 11.7 Channel-ranking accuracy

If:

\[
ROAS_{Search} > ROAS_{Meta} > ROAS_{TV}
\]

track how frequently the fitted model correctly ranks all three channels.

---

## 11.8 Total-media recovery

This is extremely important.

\[
TotalMediaError =
\hat{IncrementalRevenue}_{all media}
-
TrueIncrementalRevenue_{all media}
\]

A common phenomenon worth testing is:

> The model may estimate total media effect reasonably while misallocating effect among correlated channels.

This distinction should become a recurring theme.

---

## 11.9 Allocation error

Example:

\[
AllocationError =
\sum_m
|\hat{Contribution}_m - TrueContribution_m|
\]

This directly measures whether the model distributes the total media effect correctly.

---

## 11.10 Model fit

Track at least:

- R²
- MAPE
- weighted MAPE where available
- train metrics
- holdout/test metrics

Do not make R² the main success criterion.

The project is partly designed to show that predictive fit and causal attribution can diverge.

Current Meridian tooling exposes R², MAPE and weighted MAPE for model-fit assessment.

---

## 11.11 Bayesian diagnostics

For Meridian save:

- R-hat
- convergence health checks
- prior–posterior shift results
- high-variance checks
- implausible ROI checks
- baseline checks
- posterior predictive checks where available

The current Meridian reviewer includes model-quality checks for convergence, baseline, Bayesian posterior predictive behaviour, goodness of fit, high variance, implausible ROI, potential bias, prior-posterior shift and ROI consistency.

---

# 12. Master results schema

Save every fitted model to a tidy result table.

Example:

```text
experiment
scenario
simulation_id
seed
n_weeks
channel
true_roas
posterior_mean_roas
posterior_median_roas
ci_low
ci_high
relative_error
covered_truth
true_contribution
estimated_contribution
r2_train
r2_test
mape_train
mape_test
rhat_max
prior_mean
prior_sd
actual_channel_correlation
media_share_of_revenue
notes
```

This allows cross-experiment analysis later.

---

# 13. Repository structure

Recommended GitHub structure:

```text
mmm-stress-tests/
│
├── README.md
├── LICENSE
├── pyproject.toml
├── requirements.txt
├── .gitignore
├── .python-version
│
├── configs/
│   ├── base.yaml
│   ├── exp01_multicollinearity.yaml
│   ├── exp02_always_on.yaml
│   └── ...
│
├── src/
│   └── mmm_stress/
│       ├── __init__.py
│       ├── dgp.py
│       ├── spend_generation.py
│       ├── transformations.py
│       ├── ground_truth.py
│       ├── meridian_runner.py
│       ├── simple_models.py
│       ├── metrics.py
│       ├── plotting.py
│       └── utils.py
│
├── notebooks/
│   ├── 00_validate_dgp.ipynb
│   ├── 01_multicollinearity.ipynb
│   ├── 02_always_on.ipynb
│   ├── 03_prior_sensitivity.ipynb
│   ├── 04_endogeneity.ipynb
│   ├── 05_low_media_signal.ipynb
│   ├── 06_omitted_confounder.ipynb
│   ├── 07_baseline_flexibility.ipynb
│   ├── 08_adstock_misspecification.ipynb
│   ├── 09_saturation_misspecification.ipynb
│   ├── 10_sample_size.ipynb
│   ├── 11_noise.ipynb
│   ├── 12_holdout.ipynb
│   ├── 13_experiment_calibration.ipynb
│   └── 14_combined_real_world.ipynb
│
├── data/
│   ├── synthetic/
│   └── README.md
│
├── results/
│   ├── raw/
│   ├── summaries/
│   └── model_objects/
│
├── figures/
│   ├── linkedin/
│   ├── technical/
│   └── videos/
│
├── posts/
│   ├── exp01_linkedin.md
│   ├── exp01_video_script.md
│   └── ...
│
└── tests/
    ├── test_adstock.py
    ├── test_ground_truth.py
    ├── test_spend_generation.py
    └── test_metrics.py
```

---

# 14. Configuration-first design

Do not hard-code every experiment in notebooks.

Use YAML.

Example:

```yaml
experiment:
  id: exp01
  name: multicollinearity

simulation:
  n_weeks: 156
  n_sims: 100
  seed: 20260831
  noise_sd: 120000

channels:
  meta:
    true_roas: 2.0
    mean_spend: 55000
    spend_sd: 14000

  search:
    true_roas: 5.0
    mean_spend: 42000
    spend_sd: 11000

  tv:
    true_roas: 1.5
    mean_spend: 90000
    spend_sd: 45000

stress:
  correlation:
    meta_search:
      - 0.20
      - 0.50
      - 0.70
      - 0.90
      - 0.95
      - 0.98
```

This will make the repository look much more professional.

---

# 15. Experiment 00 — Clean control / recoverability

## Question

> Can the model recover known ROAS when the data is intentionally favourable?

## Purpose

This is the control experiment.

Do not skip it.

If Meridian cannot recover the DGP under favourable conditions, subsequent stress tests are uninterpretable.

## DGP

Use:

- 156 weeks
- 3 media channels
- low channel correlation
- substantial spend variation
- all confounders observed
- no endogeneity
- no omitted variables
- moderate noise
- linear media initially
- correct model specification

True ROAS:

```text
Meta   = 2.0
Search = 5.0
TV     = 1.5
```

Target correlations:

```text
Meta/Search < 0.2
Meta/TV     < 0.2
Search/TV   < 0.2
```

## Hypothesis

Posterior estimates should recover true ROAS reasonably well.

## Success criteria

Suggested:

- median relative ROAS error < 20%
- 90% credible interval coverage close to nominal
- correct channel ranking in most simulations
- total-media recovery accurate
- no meaningful convergence problems

## Figures

1. True vs estimated ROAS
2. 90% intervals vs truth
3. true vs estimated contributions
4. model-fit plot
5. prior vs posterior distributions

## Public lesson

> Before trying to break MMM, prove it can recover a known answer under clean conditions.

---

# 16. Experiment 01 — Multicollinearity

## Question

> What happens to channel attribution when two media channels increasingly move together?

## Stress variable

Increase correlation between Meta and Search:

```python
RHO_LEVELS = [
    0.0,
    0.2,
    0.5,
    0.7,
    0.8,
    0.9,
    0.95,
    0.98,
]
```

Keep TV independent.

## Spend-generation approach

Generate correlated latent shocks:

```python
cov = np.array([
    [1.0, rho],
    [rho, 1.0],
])

z = rng.multivariate_normal(
    mean=[0, 0],
    cov=cov,
    size=n_weeks,
)

meta_spend = meta_mean + meta_sd * z[:, 0]
search_spend = search_mean + search_sd * z[:, 1]
```

Clip spend at a small positive number.

## Keep constant

- true ROAS
- baseline
- noise
- time length
- priors
- transformations
- promotions
- TV behaviour

## Hypotheses

As correlation rises:

1. channel-specific ROAS intervals widen
2. channel allocation becomes unstable
3. total-media effect may remain more stable
4. predictive fit may remain strong
5. posterior estimates may become more prior-sensitive

## Metrics

- actual realised correlation
- ROAS bias
- >25% / >50% error probability
- ranking accuracy
- interval width
- interval coverage
- total-media error
- allocation error
- R² / wMAPE
- prior–posterior shift

## Key chart

X-axis:

```text
Correlation between Meta and Search
```

Y-axis:

```text
Probability estimated ROAS is >50% from truth
```

Secondary chart:

```text
Correlation vs 90% credible interval width
```

## Video hook

> I gave an MMM two channels with different true ROAS — then slowly made their spend patterns identical.

## Core message

> Multicollinearity may damage identification much more than prediction.

## Important nuance

Do not say:

> "Multicollinearity automatically biases coefficients."

In a correctly specified classical linear model, multicollinearity primarily inflates variance rather than mechanically introducing bias.

The stronger finding is:

> Individual model runs become less reliable and attribution becomes weakly identified.

---

# 17. Experiment 02 — Always-on channels / weak variation

## Question

> Can MMM measure the effect of a channel whose spend barely changes?

## Setup

Search true ROAS:

```text
5.0
```

Start with high variability:

```python
CV = spend_sd / spend_mean
```

Stress levels:

```text
CV = 0.50
CV = 0.30
CV = 0.20
CV = 0.10
CV = 0.05
CV = 0.02
```

Alternative representation:

```python
search_spend =
    mean_spend * (1 + variation_scale * random_shock)
```

## Keep mean spend constant

This matters.

You want variation—not average investment—to be the manipulated variable.

## Hypothesis

As spend variation approaches zero:

- Search effect becomes weakly identified
- credible intervals widen
- prior influence grows
- model may still fit total revenue well
- estimated Search ROAS may remain superficially plausible despite weak information

## Additional metric

Track:

```text
coefficient of variation of channel spend
```

and:

```text
effective number of distinct spend states
```

## Public message

> A model cannot learn a response curve from a channel that barely moves.

---

# 18. Experiment 03 — Prior sensitivity

## Question

> When does the data overcome the prior — and when does the prior effectively choose the answer?

## Run under two identification regimes

### Regime A: strong data

- low channel correlation
- good variation
- long time series
- decent media signal

### Regime B: weak data

- high correlation OR always-on behaviour
- everything else identical

## True ROAS

```text
Meta = 2.0
```

## Prior scenarios

Example conceptual priors:

```text
Pessimistic prior:
mean/median around 1

Correct-ish prior:
around 2

Optimistic prior:
around 5

Very optimistic prior:
around 8
```

Use proper lognormal parameter conversion rather than passing arithmetic means directly as log-space parameters.

## Meridian

Current Meridian supports treatment prior types including:

- ROI
- mROI
- contribution
- coefficient

For paid media, ROI is usually the easiest business-facing prior for this series.

## Hypothesis

Strong identification:

```text
different priors -> similar posteriors
```

Weak identification:

```text
different priors -> meaningfully different posteriors
```

## Metrics

For each prior:

- prior mean
- prior SD
- posterior mean
- posterior SD
- posterior median
- true ROAS
- prior-to-posterior movement
- error from truth
- R²
- likelihood/model fit
- ranking

## Killer visual

Three or four posterior distributions from the same dataset:

```text
Prior centred at 1
Prior centred at 2
Prior centred at 5
Prior centred at 8
```

Overlay vertical line:

```text
TRUE ROAS = 2
```

## Core message

> Bayesian priors are not inherently a problem. The question is how much information the likelihood contains relative to the prior.

---

# 19. Experiment 04 — Endogeneity / spend follows demand

## Question

> What happens when marketers increase budgets precisely when sales were already going to rise?

This is one of the most important experiments.

## Create latent demand

\[
Demand_t =
Seasonality_t
+
Trend_t
+
Promo_t
+
LatentDemandShock_t
\]

Then spend reacts to demand:

\[
Spend_{Meta,t}
=
BaseSpend
+
\gamma Demand_t
+
u_t
\]

Revenue is generated from both:

\[
Revenue_t =
Baseline(Demand_t)
+
TrueMediaEffect_t
+
\epsilon_t
\]

The MMM does not observe the latent demand shock.

## Endogeneity strength

Vary:

```text
gamma = 0.0
gamma = 0.2
gamma = 0.4
gamma = 0.6
gamma = 0.8
```

Better: report the realised correlation between spend and latent demand.

## Two sub-experiments

### 04A — Endogeneity with omitted demand proxy

MMM does not observe latent demand.

### 04B — Add imperfect demand controls

Examples:

- generic search query volume
- site traffic proxy
- category demand index

Construct:

\[
DemandProxy_t =
Demand_t + MeasurementNoise_t
\]

Test proxy quality:

```text
corr(proxy, latent demand)
= 0.3 / 0.5 / 0.7 / 0.9
```

## Questions

- How badly does omitted demand bias ROAS?
- How good must a proxy be before it meaningfully helps?
- Can controls introduce their own problems?
- Does predictive fit reveal the causal failure?

## Core message

> Spend–sales correlation is not the same thing as advertising causality.

---

# 20. Experiment 05 — Weak media signal

## Question

> What happens when media explains only a small fraction of total revenue?

## Scenarios

Set media contribution to approximately:

```text
40%
30%
20%
10%
5%
2%
```

Do this by scaling baseline magnitude while preserving media effects, or vice versa.

Be explicit about which quantity is held fixed.

## Recommended method

Keep true media ROAS fixed.

Increase baseline revenue.

This maintains a consistent causal media mechanism while reducing media's share of observed KPI variation.

## Hypothesis

As media signal shrinks:

- posterior uncertainty increases
- priors matter more
- contribution recovery worsens
- individual channel attribution becomes fragile
- predictive fit may remain very strong because baseline dominates

## Key chart

X-axis:

```text
True media share of revenue
```

Y-axis:

```text
Median absolute ROAS error
```

Add secondary panel/chart:

```text
R²
```

## Core message

> A high-fidelity sales forecast can coexist with very little information about marketing incrementality.

---

# 21. Experiment 06 — Omitted confounders

## Question

> How much can one missing business driver distort media attribution?

## DGP

Generate a promotion variable:

\[
Promo_t \in \{0,1\}
\]

Promotions increase:

1. revenue
2. media spend

Example:

\[
Revenue_t += \delta Promo_t
\]

and:

\[
MetaSpend_t += \eta Promo_t
\]

Then fit:

### Model A

Includes Promo.

### Model B

Omits Promo.

## Stress level

Vary promotion effect and media-promo correlation.

## Extensions

Repeat using:

- price reductions
- distribution changes
- competitor activity
- weather
- product launches
- category-demand shocks

## Key result

Compare:

```text
true ROAS
estimated ROAS with confounder
estimated ROAS without confounder
```

## Core message

> A variable does not need to be "marketing" to materially change marketing attribution.

---

# 22. Experiment 07 — Baseline flexibility

## Question

> Can an overly flexible baseline absorb true media effects — or can an inflexible baseline force baseline movement into media?

This is a crucial but under-discussed problem.

## Synthetic baseline

Create:

- smooth trend
- annual seasonality
- medium-frequency demand movement
- promotions

## Model specifications

Fit increasingly flexible time effects / knot choices.

Conceptually:

```text
Very rigid
Moderate
Correct complexity
Highly flexible
Near-saturated baseline
```

## Two possible failure directions

### Too rigid

Baseline cannot explain genuine non-media movement.

Media may absorb it.

### Too flexible

Baseline may absorb genuine media-driven movement.

## Metrics

- true baseline vs estimated baseline
- total baseline share
- media contribution bias
- channel ROAS bias
- model fit
- holdout fit

## Key visual

Overlay:

```text
true baseline
estimated baseline
```

Then show:

```text
true total media contribution
estimated total media contribution
```

## Core message

> Baseline specification is part of the attribution model, not just a nuisance component.

---

# 23. Experiment 08 — Adstock misspecification

## Question

> What happens when the model assumes the wrong carryover duration?

## True adstock

Example:

```text
Meta  alpha = 0.30
Search alpha = 0.10
TV    alpha = 0.70
```

## Fit scenarios

### Correct

Model can represent the true decay.

### Under-carryover

Force/strongly encourage too-short lag/decay.

### Over-carryover

Force/strongly encourage too-long carryover.

## Meridian notes

Current Meridian supports:

- geometric decay
- binomial decay
- channel-specific adstock specifications
- configurable max lag

## Metrics

- recovered ROI
- recovered adstock parameters
- incremental time-path recovery
- response delay
- total contribution
- holdout prediction

## Important distinction

Even if total contribution is correct, timing can be wrong.

Measure:

\[
TimePathRMSE =
RMSE(
TrueIncremental_t,
EstimatedIncremental_t
)
\]

## Core message

> Getting the total effect roughly right does not guarantee the model understands when that effect occurred.

---

# 24. Experiment 09 — Saturation misspecification

## Question

> Can MMM recover ROI when the real response is nonlinear but the fitted model assumes the wrong shape?

## True DGP

Use Hill saturation.

## Scenarios

### A

Generate Hill, fit Hill.

### B

Generate Hill, fit no saturation / linear.

### C

Generate almost-linear response, fit strong nonlinear saturation.

## Spend-range manipulation

This is essential.

Test:

```text
Observed spend covers wide part of response curve
```

versus:

```text
Observed spend stays in a narrow local region
```

The second case may make saturation parameters impossible to identify.

## Metrics

- historical average ROAS
- mROAS at current spend
- response-curve RMSE
- optimal budget error
- extrapolation error

## Optimisation extension

Generate the true optimal allocation from the DGP.

Compare:

```text
true optimal budget
MMM-recommended budget
```

This makes the experiment commercially relevant.

## Core message

> A model can estimate historical contribution reasonably while still learning the wrong response curve for optimisation.

---

# 25. Experiment 10 — Sample size / time-series length

## Question

> How much history does MMM need before channel estimates become reasonably stable?

## Scenarios

```text
52 weeks
78 weeks
104 weeks
130 weeks
156 weeks
208 weeks
```

Keep the underlying process identical.

## Important

The number of observations alone is not enough.

Also report:

```text
number of independent spend changes
number of campaigns/bursts
number of seasonal cycles
```

## Metrics

- ROAS interval width
- ROAS error
- ranking accuracy
- convergence
- response-curve uncertainty
- holdout performance

## Core message

> "Three years of data" is not automatically three years of useful identification.

---

# 26. Experiment 11 — Outcome noise

## Question

> How does unexplained sales volatility affect attribution?

## Noise scenarios

Scale residual standard deviation:

```text
0.25x
0.5x
1x
2x
4x
```

Also express noise relative to true media contribution.

## Optional extension

Use heteroskedastic noise:

\[
\sigma_t \propto Revenue_t
\]

or heavy-tailed shocks.

## Metrics

- ROAS RMSE
- interval width
- coverage
- convergence
- model fit
- ranking accuracy

## Core message

> More sales noise does not necessarily destroy prediction immediately, but it reduces the information available to separate small media effects.

---

# 27. Experiment 12 — Holdout performance versus attribution quality

## Question

> Does good out-of-sample prediction imply good causal attribution?

## Design

Generate datasets across several stress conditions.

For each fitted model record:

```text
test R²
test wMAPE
ROAS error
allocation error
total-media error
```

Then plot:

```text
Holdout R² vs ROAS error
```

and:

```text
Holdout wMAPE vs ROAS error
```

## Hypothesis

Prediction quality and attribution quality are related but not equivalent.

You may find models with:

```text
good test prediction + poor channel recovery
```

## Core message

> Holdout fit is necessary evidence about generalisation, but it is not a direct validation of causal decomposition.

---

# 28. Experiment 13 — Geo data versus national aggregation

## Question

> Does geo-level variation materially improve identification compared with national aggregation?

## DGP

Generate:

```text
20–50 geos
```

with:

- different media spend intensity
- shared seasonal pattern
- geo-specific intercepts
- controlled heterogeneity
- optional geo-specific media effects

Fit:

### Model A

Geo-level MMM.

### Model B

Aggregate to national level and fit national MMM.

## Questions

- Are intervals tighter at geo level?
- Does channel ranking improve?
- Does geo variation help identify saturation?
- What happens if geos are heterogeneous beyond model assumptions?

Current Meridian guidance notes that geo-level data can increase effective sample size, provide more spend variation, improve time-effect estimation, and potentially reduce omitted-variable bias.

## Core message

> More rows are not the only benefit of geo MMM; cross-sectional media variation may create identification that national aggregation removes.

---

# 29. Experiment 14 — Experiment-informed prior calibration

## Question

> When does an external incrementality experiment improve MMM — and when can calibration mislead it?

## Step 1

Generate synthetic MMM data with true:

```text
Meta ROAS = 2.0
```

## Step 2

Generate a simulated experiment.

Example experiment estimates:

```text
Estimate = 2.1
SE       = 0.4
```

## Step 3

Fit:

### A

Generic/default prior.

### B

Experiment-informed prior centred near 2.1.

### C

Wrong/stale experiment prior centred at 4.0.

### D

Overconfident wrong prior centred at 4.0 with tiny uncertainty.

## Stress dimensions

Experiment may differ in:

- time window
- geography
- spend level
- campaign mix
- audience
- marginal vs zero-spend estimand

Current Meridian documentation explicitly cautions that experiment ROI and MMM ROI can target different estimands and recommends accounting for this additional uncertainty when using experiment results as priors.

## Metrics

- posterior ROAS accuracy
- interval coverage
- prior–posterior shift
- total-media accuracy
- ranking
- calibration sensitivity

## Core message

> Experiment calibration is powerful, but an external measurement should inform the MMM rather than be treated as unquestionable truth.

---

# 30. Experiment 15 — Combined realistic stress test

Only do this after the individual experiments.

## Question

> What happens when a synthetic dataset starts to resemble messy real marketing data?

Combine:

- 156 weeks
- Meta/Search correlation ~0.85
- Search CV ~0.08
- media = ~10% of revenue
- spend partially follows latent demand
- promotion control observed
- demand proxy imperfect
- true adstock
- true saturation
- noisy revenue
- realistic ROI priors
- one omitted confounder

## Model variants

### Model A — naive

Basic MMM.

### Model B — better controls

Add demand proxy.

### Model C — stronger priors

Use calibrated ROI information.

### Model D — aggregated correlated channels

Combine channels where justified.

### Model E — experiment-calibrated

Use external lift evidence.

## Evaluate

- individual channel recovery
- total-media recovery
- baseline recovery
- response curves
- optimisation recommendation
- predictive fit
- uncertainty
- prior dependence

## Final series message

> MMM reliability is not one property of one model. It depends on data variation, confounding, signal strength, assumptions, priors, and external evidence.

---

# 31. Optional Experiment 16 — Channel aggregation

## Question

> When two channels cannot be separately identified, is it better to estimate their combined effect?

Generate two highly correlated channels.

True:

```text
Meta ROAS   = 2
Search ROAS = 5
```

Compare:

### Model A

Separate channels.

### Model B

Aggregate:

```text
Performance = Meta + Search
```

Ground-truth combined incremental revenue is known.

## Evaluation

Compare:

- total contribution error
- uncertainty
- stability
- usefulness for budget decisions
- information lost by aggregation

## Core message

> A less granular answer can sometimes be more identifiable than a detailed but unstable answer.

---

# 32. Optional Experiment 17 — Reverse causality / brand search

## Question

> What happens when sales demand itself creates channel activity?

This is especially relevant for branded paid search.

Construct latent demand:

\[
Demand_t
\]

Then:

\[
BrandSearch_t
=
a + b \times Demand_t + noise
\]

while Brand Search also has some true causal effect:

\[
Revenue_t += \beta BrandSearch_t
\]

Now Brand Search is simultaneously:

- a response to demand
- a cause of additional sales

This produces a realistic simultaneity problem.

Compare:

- no controls
- generic-query demand proxy
- organic brand search proxy
- lagged demand proxy
- channel aggregation

## Core message

> Branded search may be both a thermometer and a heater.

---

# 33. Optional Experiment 18 — Measurement error in media spend/exposure

## Question

> What happens when the MMM input itself is measured poorly?

Generate true exposure/spend.

Then observed media is:

\[
ObservedSpend =
TrueSpend + MeasurementError
\]

Stress:

```text
0%
5%
10%
20%
40%
```

Potential examples:

- incomplete platform data
- attribution-window changes
- missing offline spend
- estimated impressions
- reporting changes

## Core message

> MMM can only identify effects from the media signal you actually provide to it.

---

# 34. The DGP validation notebook

Before fitting any MMM, create:

```text
00_validate_dgp.ipynb
```

It should verify:

## Baseline

- expected trend
- seasonality
- promotion effect
- mean revenue
- variance

## Media

- average spend
- spend CV
- correlations
- true incremental revenue
- true ROAS
- media share of revenue

## Transformations

- adstock behaves as expected
- Hill curve has correct shape
- counterfactual zero-media calculation reproduces true effect

## Reproducibility

Run same seed twice and assert identical output.

---

# 35. Unit tests

At minimum:

## test_ground_truth.py

Test that:

```python
incremental = revenue_with_media - revenue_without_media
```

matches stored ground truth.

## test_adstock.py

Known input:

```text
[100, 0, 0, 0]
```

with alpha = 0.5 should produce:

```text
[100, 50, 25, 12.5]
```

for the recursive formulation used.

## test_correlation_generation.py

Generated correlation should be near target over sufficiently large samples.

## test_zero_media.py

All-zero spend should produce zero incremental media contribution.

## test_seed_reproducibility.py

Same seed = same synthetic dataset.

---

# 36. Meridian implementation notes

Meridian's API evolves, so pin the version used in every experiment.

At the time this blueprint was created, the official JAX getting-started notebook demonstrated:

```bash
pip install --upgrade google-meridian[colab,and-cuda,schema]
```

and showed version 1.7.0 being installed in that environment.

Do not depend on `latest` for a published reproducibility repository.

Instead pin:

```text
google-meridian==<version-you-used>
```

and record:

```text
Python version
Meridian version
JAX/TensorFlow backend
GPU type
OS
Git commit
```

---

# 37. Current Meridian model-spec features useful for this series

The current documentation exposes model-spec controls including:

```text
media_prior_type
rf_prior_type
roi_calibration_period
max_lag
adstock_decay_spec
saturation_spec
holdout_id
enable_aks
```

The currently documented adstock options include:

```text
geometric
binomial
```

and saturation options include:

```text
hill
none
```

These may be provided by channel.

Use the official documentation rather than relying on copied code from old blog posts because the API has changed over time.

---

# 38. Suggested Meridian workflow per scenario

Pseudocode:

```python
# 1. generate synthetic dataset
df, truth = generate_dataset(config)

# 2. convert to Meridian input format
data = build_meridian_input(df, config)

# 3. define priors
prior = build_prior(config)

# 4. define model spec
model_spec = spec.ModelSpec(
    prior=prior,
    media_prior_type="roi",
    max_lag=config.max_lag,
    adstock_decay_spec=config.adstock_spec,
    saturation_spec=config.saturation_spec,
    holdout_id=holdout_id,
)

# 5. instantiate
mmm = model.Meridian(
    input_data=data,
    model_spec=model_spec,
)

# 6. sample prior
mmm.sample_prior(...)

# 7. sample posterior
mmm.sample_posterior(...)

# 8. health checks
health = reviewer.ModelReviewer(mmm).run()

# 9. extract posterior ROAS / contribution
posterior = extract_results(mmm)

# 10. compare to ground truth
metrics = evaluate_recovery(
    posterior=posterior,
    truth=truth,
)

# 11. save model + tidy results
```

Adapt function names to the exact Meridian version in your environment.

---

# 39. MCMC settings

Do not optimise for speed before validating convergence.

During development:

```text
fewer chains / draws may be acceptable
```

For final published results:

- use multiple chains
- enough adaptation/burn-in
- enough posterior draws
- inspect convergence
- save R-hat
- repeat problematic scenarios

If an experiment's conclusion depends on unconverged models, do not publish it as an MMM finding.

Instead publish:

> "This scenario caused the sampler/model to struggle."

That itself may be informative, but it is different from a causal-recovery result.

---

# 40. Prior discipline

For every experiment, save the exact prior.

Never describe it merely as:

```text
weak
strong
optimistic
```

Record:

```text
distribution
parameterisation
mean
median
SD
5th percentile
95th percentile
```

Plot it.

For lognormal priors, remember that log-space parameters are not the same as arithmetic mean and standard deviation.

Use helper functions where available or explicitly convert parameters.

---

# 41. A critical rule for prior experiments

Do not compare priors while silently changing other settings.

Same:

- dataset
- seed
- likelihood
- baseline
- adstock
- saturation
- MCMC settings

Only prior changes.

This creates a clean sensitivity experiment.

---

# 42. Counterfactual truth with adstock

When calculating true incremental revenue, be precise.

If media is set to zero during a period, previous media may still create carryover.

Define your estimand explicitly.

Possible estimands:

## Full-history zero-media effect

Set the channel to zero for the whole modelling horizon.

## Period-specific spend removal

Set spend to zero only for selected weeks while retaining prior carryover.

## Meridian-aligned calibration estimand

When comparing directly with Meridian ROI definitions, align your synthetic truth to Meridian's documented ROI counterfactual as closely as possible.

This is particularly important for experiment-calibration work.

---

# 43. Media spend versus exposure

Decide whether:

```text
media == spend
```

for early experiments.

That is acceptable for a simple synthetic series.

Later, make them separate:

\[
Exposure_t = f(Spend_t, CPM_t)
\]

with varying media cost.

This allows another future stress test:

> Can changing CPMs confuse spend-based interpretations?

---

# 44. Holdout design

Do not randomly hold out individual weeks if the production use case is forecasting future time.

Prefer contiguous final-period holdout:

```text
last 10–20% of weeks
```

Example:

```python
N_HOLDOUT = 16
```

Also test rolling holdouts later.

Save:

- train R²
- test R²
- train wMAPE
- test wMAPE
- attribution recovery

---

# 45. What NOT to conclude

Avoid overclaiming.

Examples:

## Bad

> High correlation makes MMM wrong.

## Better

> In this synthetic DGP, increasing channel correlation materially increased channel-level uncertainty and recovery error while aggregate fit remained similar.

---

## Bad

> R² is useless.

## Better

> Predictive fit is useful, but it does not directly validate the causal allocation of sales across channels.

---

## Bad

> Priors manipulate the answer.

## Better

> When the likelihood contains weak information, posterior attribution can become more sensitive to prior assumptions.

---

## Bad

> Experiments are always the truth.

## Better

> Experiments provide strong causal evidence for their specific estimand and context, but translating them into MMM priors requires judgement.

---

# 46. Publication standard

For every public result include:

1. research question
2. synthetic truth
3. one manipulated variable
4. number of simulations
5. model/framework
6. prior specification
7. key result
8. uncertainty
9. caveat
10. link to code

This will make the project much more credible.

---

# 47. Core figures to reuse

Create standard plotting helpers.

## Figure A — true vs estimated ROAS

For each channel:

```text
true value
posterior median
90% credible interval
```

## Figure B — error versus stress

Example:

```text
correlation -> ROAS error
```

## Figure C — credible interval width

## Figure D — probability of >50% error

## Figure E — total media recovery

## Figure F — allocation error

## Figure G — model fit versus attribution error

## Figure H — prior vs posterior

## Figure I — response curves

## Figure J — baseline truth vs estimate

Use consistent dimensions and formatting across the series.

---

# 48. LinkedIn chart rule

Technical figures can be complex.

LinkedIn figures should answer one question in three seconds.

Prefer:

```text
ONE headline
ONE x-axis
ONE y-axis
ONE key takeaway
```

Avoid:

- 12 legends
- tiny posterior panels
- notebook screenshots
- large tables
- dense diagnostics

Put deeper diagnostics on GitHub.

---

# 49. Experiment page template

Create one README/Markdown page per experiment.

Template:

```markdown
# MMM Stress Test XX — [Name]

## Question

## Why this matters

## Hypothesis

## Ground truth

## Data-generating process

## Stress variable

## Conditions held constant

## Model specification

## Priors

## Simulation grid

## Evaluation metrics

## Results

## Main figure

## Secondary diagnostics

## Interpretation

## What this does NOT prove

## Limitations

## Reproduction instructions

## Next experiment
```

---

# 50. LinkedIn post template

Use roughly:

```text
HOOK

I deliberately gave an MMM [specific problem].

Because I generated the data myself, I knew the true answer:

Meta ROAS = X
Search ROAS = Y
TV ROAS = Z

I changed only one thing:

[stress variable]

I ran the experiment [N] times.

At [low stress]:
[result]

At [high stress]:
[result]

The interesting part:

[model fit/aggregate contribution remained...]

My takeaway:

[one careful conclusion]

This does not mean [overclaim].

It means [precise interpretation].

Code + methodology: GitHub link
```

---

# 51. Short-video template

Target:

```text
45–90 seconds
```

## 0–4 sec — hook

> I tried to break a Media Mix Model.

## 4–12 sec — truth

Show:

```text
TRUE ROAS
Meta   2.0
Search 5.0
TV     1.5
```

Say:

> I generated the dataset, so I know the causal answer.

## 12–25 sec — manipulation

Show one variable changing.

Example:

```text
corr(Meta, Search)
0.2 -> 0.95
```

## 25–45 sec — result

Animate or reveal chart.

## 45–65 sec — surprise

Example:

> Model fit barely moved, but channel-level error exploded.

## 65–80 sec — interpretation

> The model still explained revenue. It simply had much less information about how to divide credit between those channels.

## 80–90 sec — next test

> Next I am going to keep the same dataset and change only the priors.

---

# 52. Video production style

For this series, screen-led videos are more useful than talking-head-only videos.

Recommended visual sequence:

1. your face for the hook, optional
2. synthetic truth card
3. 5–10 seconds of notebook/code
4. simple chart
5. one conclusion card

Do not show long blocks of code.

Show only the line that creates the stress:

```python
rho = 0.95
```

or:

```python
meta_spend = base + gamma * latent_demand + noise
```

That makes the experiment visually understandable.

---

# 53. One experiment = multiple pieces of content

Each experiment should create:

## Asset 1

Technical notebook.

## Asset 2

GitHub experiment README.

## Asset 3

Main LinkedIn chart.

## Asset 4

LinkedIn text post.

## Asset 5

Short video.

## Asset 6

Optional technical article/newsletter.

## Asset 7

Later: compilation into a larger report.

This dramatically improves return on research effort.

---

# 54. Naming system

Use consistent names:

```text
MMM Stress Test #00 — Can MMM recover the truth?
MMM Stress Test #01 — Multicollinearity
MMM Stress Test #02 — Always-on Media
MMM Stress Test #03 — Prior Sensitivity
MMM Stress Test #04 — Endogeneity
MMM Stress Test #05 — Weak Media Signal
MMM Stress Test #06 — Omitted Confounders
MMM Stress Test #07 — Baseline Flexibility
MMM Stress Test #08 — Adstock Misspecification
MMM Stress Test #09 — Saturation Misspecification
MMM Stress Test #10 — Sample Size
MMM Stress Test #11 — Outcome Noise
MMM Stress Test #12 — Holdout Fit vs Attribution
MMM Stress Test #13 — Geo vs National
MMM Stress Test #14 — Experiment Calibration
MMM Stress Test #15 — The Real-World Gauntlet
```

---

# 55. Recommended order of publication

Do not publish in pure mathematical difficulty order.

Use audience interest.

Recommended public order:

```text
00 Clean recovery
01 Multicollinearity
03 Prior sensitivity
04 Endogeneity
02 Always-on media
05 Weak media signal
06 Omitted confounders
12 Fit vs attribution
08 Adstock
09 Saturation
10 Data length
07 Baseline flexibility
14 Experiment calibration
13 Geo vs national
15 Combined realistic test
```

The first four form a strong narrative:

```text
Can it recover truth?
What if channels move together?
What if priors change?
What if spend follows demand?
```

---

# 56. Recommended first GitHub milestone

Version 0.1 should contain only:

```text
DGP
ground-truth engine
simple linear sanity model
Meridian runner
Experiment 00
Experiment 01
shared metrics
shared plots
tests
README
```

Do not build all 15 experiments before publishing anything.

Build a solid core architecture.

Then add experiments one by one.

---

# 57. README opening draft

Suggested repository opening:

> # MMM Stress Tests
>
> Media Mix Models are usually evaluated on real-world data where the true incremental effect of marketing is unknown.
>
> This project takes a different approach.
>
> I generate synthetic marketing datasets where the causal contribution and ROAS of every channel are known in advance. I then deliberately introduce common MMM problems — multicollinearity, weak variation, endogeneity, prior sensitivity, low signal, omitted confounders and model misspecification — and test whether the model can recover the truth.
>
> The goal is not to prove that MMM works or does not work.
>
> The goal is to understand the conditions under which its attribution is well identified, uncertain, prior-sensitive or misleading.

---

# 58. GitHub badges worth adding later

Possible badges:

```text
Python version
License
Tests
Code style
Meridian version
```

Only add CI badges after CI actually exists.

---

# 59. License

If you want the project widely used:

```text
MIT
```

or:

```text
Apache-2.0
```

are common options.

Choose deliberately.

Do not copy Google's license headers into your own original files unless the code itself is copied/modified from licensed Google code and the license requires preservation.

---

# 60. Citation file

Add:

```text
CITATION.cff
```

once the project becomes substantive.

This helps people cite the repository.

---

# 61. Environment reproducibility

At minimum create:

```text
requirements.txt
```

Better:

```text
pyproject.toml
```

and lock dependencies.

Record exact package versions used for published results.

For GPU Meridian runs, consider providing a Colab notebook as well as local setup.

---

# 62. Results immutability

For every published post:

Save the exact:

```text
config
dataset seed
model version
results CSV
figure
commit hash
```

Do not overwrite old results when code changes.

Suggested:

```text
results/exp01/2026-09-xx/
```

---

# 63. Computational strategy

Meridian Monte Carlo experiments can become expensive.

Use tiers.

## Tier 1 — DGP debugging

OLS / deterministic calculations.

Hundreds or thousands of simulations.

## Tier 2 — reduced Bayesian model

Validate qualitative behaviour cheaply.

## Tier 3 — Meridian

Run smaller but defensible grids.

Example:

```text
6 correlation levels
30 simulations each
= 180 Meridian fits
```

If too expensive:

```text
4 levels × 20 simulations
= 80 fits
```

Then increase runs for the most interesting region.

---

# 64. Avoid p-hacking the content

Pre-register the hypothesis inside the experiment Markdown before running the final simulation.

Example:

```markdown
## Pre-run hypothesis

I expect correlation to increase uncertainty and channel allocation error, while total-media recovery and model fit deteriorate less rapidly.
```

Commit that before seeing final results if practical.

This makes the GitHub project more scientifically credible.

---

# 65. Report negative results

If an experiment does not produce the expected failure:

Publish it.

Example:

> I expected high correlation to destroy total media recovery. It didn't.

That may be more interesting than confirming your prior belief.

The project should investigate MMM, not manufacture criticism of MMM.

---

# 66. Sensitivity versus robustness

Use these terms consistently.

## Robust

Result remains similar when reasonable assumptions change.

## Sensitive

Result changes materially when an assumption changes.

## Unidentified / weakly identified

Data do not strongly distinguish among different parameter values/explanations.

## Biased

Repeated estimator systematically misses ground truth.

These are not interchangeable.

---

# 67. "Recovering truth" thresholds

Avoid one arbitrary threshold.

Report several:

```text
within 10%
within 25%
within 50%
within 100%
```

Then readers can judge practical usefulness.

---

# 68. Decision-focused evaluation

Later in the series, evaluate whether estimation error changes the business decision.

Example:

True optimal budget:

```text
Meta   £400k
Search £700k
TV     £300k
```

Model recommends:

```text
Meta   £800k
Search £350k
TV     £250k
```

Then calculate regret:

\[
DecisionRegret
=
TrueOutcome(TrueOptimalBudget)
-
TrueOutcome(ModelRecommendedBudget)
\]

This may be more important than coefficient error.

---

# 69. Optimisation regret

For experiments with saturation:

\[
Regret =
Y_{true}(Budget^*)
-
Y_{true}(\hat{Budget}_{MMM})
\]

Report:

```text
£ revenue lost
% of achievable incremental revenue lost
```

This turns the project from statistical diagnostics into marketing decision science.

---

# 70. Suggested master research questions

At the end of the full series, you should be able to answer:

1. How much spend variation is enough?
2. At what channel correlation does allocation become practically unreliable?
3. Does total-media effect remain identifiable when channels are not?
4. When do priors materially determine posterior attribution?
5. How strongly does demand-driven spend bias ROI?
6. How helpful are imperfect demand proxies?
7. How does media share of revenue affect recovery?
8. How damaging is one omitted business driver?
9. How sensitive are results to baseline flexibility?
10. Can wrong adstock assumptions preserve ROAS but distort timing?
11. Can wrong saturation assumptions produce bad optimisation?
12. How much historical data is enough?
13. Does holdout prediction correlate with causal recovery?
14. How much does geo variation improve identification?
15. When does experiment calibration rescue weak MMM?
16. What happens when all the problems coexist?
17. Which diagnostics actually warn us before attribution fails?

---

# 71. The final meta-analysis

Once all experiments are complete, pool the results.

Build a model predicting:

```text
ROAS recovery error
```

from:

```text
channel correlation
spend CV
media share
n_weeks
noise ratio
prior distance from truth
endogeneity strength
baseline flexibility
adstock misspecification
saturation misspecification
```

This could become the most valuable piece of the entire project.

Possible output:

> Which data conditions matter most for MMM identifiability?

You could even create an **MMM Reliability Map**.

---

# 72. MMM Reliability Map concept

Example dimensions:

X-axis:

```text
Channel correlation
```

Y-axis:

```text
Media signal strength
```

Colour:

```text
Probability of recovering ROAS within 25%
```

Additional maps:

```text
Spend CV × correlation
Endogeneity × demand-proxy quality
Prior strength × data strength
History length × media signal
```

This could become a recognisable visual asset associated with the series.

---

# 73. Potential long-form paper/report

Working title:

> **When Can We Trust MMM Attribution? A Synthetic Stress-Test Study of Marketing Mix Model Identifiability**

Possible sections:

1. Introduction
2. Simulation framework
3. Clean recovery
4. Multicollinearity
5. Variation
6. Priors
7. Endogeneity
8. Signal strength
9. Omitted confounders
10. Dynamic misspecification
11. Prediction vs attribution
12. Experiment calibration
13. Combined stress test
14. Practical recommendations
15. Limitations

This can later become:

- newsletter report
- conference talk
- webinar
- downloadable PDF
- portfolio project
- GitHub flagship repository

---

# 74. Limitations to state from the beginning

Synthetic experiments are powerful because truth is known.

But they are still synthetic.

Your DGP reflects assumptions.

Therefore:

> A model that succeeds on your simulation is not proven to succeed in every real business.

and:

> A model that fails under an intentionally hostile synthetic DGP is not proven to fail under every real dataset.

The strongest conclusions are conditional:

> Under this known DGP and these stress conditions, recovery behaved in this way.

---

# 75. Avoid framework wars

Do not frame this as:

```text
Meridian vs Robyn
```

at the beginning.

First understand failure modes.

Later, an optional comparison series could test:

```text
Meridian
Robyn
PyMC custom MMM
Ridge
```

on exactly the same synthetic truth.

That would be a separate project because differences in:

- priors
- transformations
- optimisation
- estimands
- regularisation
- constraints

make direct comparisons complicated.

---

# 76. What the first experiment should be

Start now with:

## Experiment 00

Clean three-channel recovery.

Then immediately:

## Experiment 01

Meta/Search correlation grid with TV independent.

Do not add saturation yet.

Do not add endogeneity yet.

Do not add always-on behaviour yet.

The first public result should be understandable without requiring the audience to trust five modelling assumptions simultaneously.

---

# 77. Exact first experiment specification

Use:

```text
Weeks: 156
Simulations per condition: start 30 Meridian runs
Channels: Meta, Search, TV

True ROAS:
Meta   2.0
Search 5.0
TV     1.5

Meta mean spend:   £55k/week
Search mean spend: £42k/week
TV mean spend:     £90k/week

Meta/Search correlation:
0.20
0.50
0.70
0.90
0.95
0.98

TV correlation:
approximately 0 with both

Controls:
trend
annual seasonality
promotions

No:
endogeneity
omitted confounders
saturation
adstock initially

Noise:
choose SD so clean case is recoverable but not deterministic

Holdout:
optional for first identification run; add after basic recovery is validated
```

---

# 78. First-experiment acceptance gate

Do not move to multicollinearity until clean case behaves sensibly.

Example gate:

```text
At rho <= 0.2:

median estimated ROAS reasonably close to truth
credible intervals cover truth at plausible rates
rankings generally correct
model converges
TV estimate stable
```

If not:

debug:

- DGP
- scale
- priors
- Meridian input
- model configuration
- sampling
- counterfactual calculation

before adding stress.

---

# 79. First public chart

Headline:

> **Same sales fit. Much less reliable channel attribution.**

X-axis:

```text
Correlation between Meta and Search spend
```

Y-axis:

```text
% of simulations with ROAS >50% from truth
```

Lines:

```text
Meta
Search
TV
```

TV should ideally remain relatively stable and act as a control.

---

# 80. First public video

Suggested script structure:

> I tried to break a Media Mix Model.
>
> I created three synthetic marketing channels, so I knew their actual causal ROAS:
>
> Meta = 2.
> Search = 5.
> TV = 1.5.
>
> Then I changed only one thing.
>
> I made Meta and Search spend increasingly correlated.
>
> At low correlation, the model could separate them reasonably well.
>
> But as the two channels started moving almost identically, the individual ROAS estimates became much less stable.
>
> The interesting part was that overall sales fit could still look good.
>
> So the model could explain sales without necessarily knowing exactly how to divide the credit between two channels.
>
> That doesn't mean multicollinearity makes MMM useless.
>
> It means channel-level attribution can become weakly identified even when the model looks healthy from a prediction perspective.
>
> Next stress test: priors.

Replace all quantitative claims with actual final Meridian results before recording.

---

# 81. First LinkedIn post title/hook options

Option A:

> **I gave an MMM two channels that moved almost identically. Then I checked whether it could still recover the true ROAS.**

Option B:

> **My MMM still fit sales beautifully. Its channel attribution became much less reliable.**

Option C:

> **How correlated can two marketing channels become before MMM stops separating them reliably?**

Option A is probably the cleanest first post.

---

# 82. Posting cadence

Research quality comes first.

Suggested:

```text
Week 1:
Experiment build + result post

Week 1/2:
short video

Week 2:
technical explainer based on same result

Week 2/3:
next experiment
```

You do not need three entirely new research experiments every week.

One good experiment can support several posts.

---

# 83. Versioning

Use tags:

```text
v0.1.0 — clean DGP + multicollinearity
v0.2.0 — always-on + priors
v0.3.0 — endogeneity
...
```

For published results, tag the repository commit.

---

# 84. Data provenance

Clearly label all synthetic data.

Never let a reader mistake synthetic results for client data.

Add to every notebook:

> **All data in this experiment are synthetic. No client or proprietary marketing data are used.**

---

# 85. Ethics / professional safety

Do not reconstruct a real client dataset too closely.

Avoid:

- exact client spend
- exact revenue
- exact channel mix
- exact dates
- identifiable promotions
- internal priors

Synthetic examples should be independently generated.

---

# 86. Suggested README disclaimer

> This project is an educational and research exercise using synthetic data. Results demonstrate the behaviour of specified models under specified simulated conditions and should not be interpreted as universal performance guarantees for MMM frameworks or as recommendations for any specific advertiser.

---

# 87. Sources and current Meridian references

The following official sources were checked when creating this blueprint.

## Meridian overview / repository

https://github.com/google/meridian

## Model specification

https://developers.google.com/meridian/reference/api/meridian/model/spec/ModelSpec

At the time of writing, this documents:

- `media_prior_type`
- `rf_prior_type`
- `max_lag`
- `holdout_id`
- `adstock_decay_spec`
- `saturation_spec`
- `roi_calibration_period`
- `enable_aks`

## Data requirements

https://developers.google.com/meridian/docs/pre-modeling/collect-data

## Priors

https://developers.google.com/meridian/docs/advanced-modeling/intro-priors

## Treatment prior types

https://developers.google.com/meridian/docs/advanced-modeling/how-to-choose-treatment-prior-types

## Experiment / ROI calibration

https://developers.google.com/meridian/docs/advanced-modeling/roi-priors-and-calibration

## Current getting-started notebook

https://developers.google.com/meridian/notebook/meridian-getting-started-jax

## Model fit

https://developers.google.com/meridian/reference/api/meridian/schema/processors/model_fit_processor

## Model diagnostics / Analyzer

https://developers.google.com/meridian/reference/api/meridian/analysis/analyzer/Analyzer

## Geo-level modelling guidance

https://developers.google.com/meridian/docs/pre-modeling/geo-selection-national-data

---

# 88. Final operating principle

At every stage ask:

> **Does this experiment tell us something about causal recoverability, or merely about prediction?**

The most interesting MMM problems occur when those two answers diverge.

The project should therefore keep three quantities separate:

```text
1. Can the model predict sales?
2. Can it recover total incremental media impact?
3. Can it correctly allocate that impact across individual channels?
```

Those are not the same problem.

If the series demonstrates that distinction carefully, reproducibly and without overclaiming, it can become a genuinely strong body of work rather than another collection of MMM opinions.

---

# 89. Immediate next actions

1. Create the GitHub repository.
2. Add this blueprint as `SERIES_BLUEPRINT.md`.
3. Build `src/mmm_stress/dgp.py`.
4. Build `src/mmm_stress/ground_truth.py`.
5. Add unit tests for truth and reproducibility.
6. Generate the Experiment 00 clean dataset.
7. Validate true ROAS and contributions manually.
8. Run a simple OLS sanity check.
9. Run Meridian on the clean case.
10. Confirm convergence and truth recovery.
11. Freeze the clean configuration.
12. Create Experiment 01 correlation grid.
13. Run Monte Carlo simulations.
14. Produce the standard scorecard.
15. Create one simple LinkedIn chart.
16. Commit the exact config/results used for the post.
17. Publish code + result together.
18. Record the short video only after the numbers are final.

---

# 90. Definition of success for the whole project

The project succeeds if, after the full series, a practitioner can look at a proposed MMM dataset and ask better questions such as:

- Do these channels have enough independent variation?
- Are we trying to estimate more granularity than the data can support?
- How much of revenue is actually attributable to media?
- Does spend react to demand?
- What important confounders are missing?
- How much are our results driven by priors?
- Do experiments support the same estimand?
- Does the baseline compete with media for the same variation?
- Are the response curves identified within observed spend ranges?
- Does good holdout prediction correspond to good causal recovery?
- Is an aggregate channel estimate more defensible than unstable channel-level estimates?

That is a much more useful outcome than simply generating a collection of high-impression posts.

