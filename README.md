# Monte Carlo Prediction Intervals — README

## Overview

This notebook builds a Monte Carlo simulation pipeline for uncertainty quantification in simple linear regression, then validates the resulting intervals with a coverage/calibration test.

**Important:** This pipeline produces **prediction intervals** (uncertainty for a new individual observation), not **confidence intervals** (uncertainty for the mean response or a parameter). See "Known Issues" below before using or citing these results.

---

## True Model

```
y = β0 + β1*x + ε,   ε ~ N(0, σ²)
```

Defaults used throughout: β0 = 1, β1 = 2, σ = 1, n = 50.

---

## Contents

**1. `generate_training_data(n, beta0, beta1, sigma)`**
Draws a fresh synthetic dataset from the true model: x ~ N(0,1), noise ~ N(0,σ²), y = β0 + β1*x + noise.

**2. `fit_linear_regression(x, y)`**
Closed-form OLS fit (no sklearn) — returns beta0_hat, beta1_hat.

**3. `predict(beta0_hat, beta1_hat, x_star)`**
Point prediction: y_hat = beta0_hat + beta1_hat * x_star.

**4. `monte_carlo_predictive_distribution(x_star, R, n, beta0, beta1, sigma)`**
Runs R (default 5000) iterations of:
- generate a fresh random training set
- fit OLS
- predict y_hat at x_star
- add one more fresh noise draw → simulated future y*

Returns an array of R simulated y* values.

**5. `monte_carlo_prediction_interval(...)`**
Wraps step 4, takes the alpha/2 and 1-alpha/2 quantiles of the simulated y* values as the interval bounds, plus the median.

**6. Histogram plot**
Visualizes the 5000 simulated y* values from one call to step 5, with dashed lines at the 2.5%, median, and 97.5% points.

**7. `coverage_test(num_experiments, confidence, R, ...)`**
Repeats the entire interval-construction process (step 5) num_experiments (default 1000) times. Each repetition:
- builds a fresh interval via Monte Carlo
- draws one independent "true" future y*
- checks whether it falls inside the interval

Returns the fraction of repetitions where it did (empirical coverage).

**8. Calibration sweep + plot**
Runs step 7 at nominal confidence levels [0.80, 0.90, 0.95, 0.99] and plots empirical vs. nominal coverage against the y=x reference line.

---

## Known Issue 1: This computes prediction intervals, not confidence intervals

Step 4 explicitly adds a fresh noise draw (`future_noise`) to each simulated value before collecting it. This makes the output a **prediction interval** (uncertainty for a new individual observation), not a **confidence interval** (uncertainty for the mean response or a parameter estimate like beta1).

- Confidence interval target: E[y | x_star] — the noiseless population regression value — or a parameter such as beta1.
- Prediction interval target: a new observation y* = E[y | x_star] + noise.

Formula-wise, the difference is one term under the square root:

```
CI:  y_hat ± t * s * sqrt(1/n + (x_star - x_bar)^2 / Sxx)
PI:  y_hat ± t * s * sqrt(1 + 1/n + (x_star - x_bar)^2 / Sxx)
```

**If confidence intervals are required:** remove the `future_noise` addition in step 4 and take quantiles of the raw y_hat values (the fitted-line predictions across resampled datasets) instead.

---

## Known Issue 2: x_train is re-randomized every iteration

In step 4, a new `x_train` (not just new noise) is generated on every Monte Carlo iteration. This means the interval marginalizes over randomness in the training design itself, not just parameter and noise uncertainty conditional on one fixed dataset. As a result, this pipeline quantifies **unconditional/total** uncertainty (as if the whole experiment — sample and all — were rerun), rather than the standard **conditional** interval (given one fixed, already-collected dataset).

**If the conditional interval is required:** generate `x_train` once, outside the loop, and only resample noise/y inside the loop.

---

## Validation Results

The coverage test confirms the intervals produced by this pipeline are well-calibrated **for the unconditional prediction-interval target actually being simulated** — empirical coverage tracks nominal coverage closely across 80/90/95/99% confidence levels. This does not confirm calibration against the classical (conditional) prediction interval formula, nor against any confidence interval target.

| Nominal | Empirical |
|---------|-----------|
| 0.80    | ~0.826    |
| 0.90    | ~0.904    |
| 0.95    | ~0.956    |
| 0.99    | ~0.991    |

---

## Suggested Fixes / Next Steps

- To get confidence intervals: drop the `future_noise` step in `monte_carlo_predictive_distribution`.
- To get the conditional (standard textbook) interval: move `x_train, y_train = generate_training_data(...)` outside the Monte Carlo loop so it's fixed across replicates, and only resample noise inside.
- Both fixes can be combined to produce the classical CI for the mean response.
