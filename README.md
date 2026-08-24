README
Overview

This notebook implements a Monte Carlo pipeline for uncertainty quantification in simple linear regression, and validates it with a coverage/calibration test.

Note: This produces prediction intervals, not confidence intervals. See "Known Issue" below.

True model

y = β₀ + β₁x + ε, ε ~ N(0, σ²)

Defaults used throughout: β₀ = 1, β₁ = 2, σ = 1, n = 50.

Contents
1. generate_training_data(n, beta0, beta1, sigma)

Draws a fresh synthetic dataset from the true model: x ~ N(0,1), noise ~ N(0,σ²), y = β₀ + β₁x + noise.

2. fit_linear_regression(x, y)

Closed-form OLS fit (no sklearn) — returns β̂₀, β̂₁.

3. predict(beta0_hat, beta1_hat, x_star)

Point prediction ŷ = β̂₀ + β̂₁·x*.

4. monte_carlo_predictive_distribution(x_star, R, n, beta0, beta1, sigma)

Runs R (default 5000) iterations of:

generate a fresh random training set
fit OLS
predict ŷ at x*
add one more fresh noise draw → simulated future y*

Returns an array of R simulated y* values.

5. monte_carlo_prediction_interval(...)

Wraps (4), takes the α/2 and 1−α/2 quantiles of the simulated y* values as the interval bounds, plus the median.

6. Histogram plot

Visualizes the 5000 simulated y* values from one call to (5), with dashed lines at the 2.5%, median, and 97.5% points.

7. coverage_test(num_experiments, confidence, R, ...)

Repeats the entire interval-construction process (5) num_experiments (default 1000) times. Each repetition:

builds a fresh interval via Monte Carlo
draws one independent "true" future y*
checks whether it falls inside the interval

Returns the fraction of repetitions where it did (empirical coverage).

8. Calibration sweep + plot

Runs (7) at nominal confidence levels [0.80, 0.90, 0.95, 0.99] and plots empirical vs. nominal coverage against the y=x reference line.

Known Issue: this computes prediction intervals, not confidence intervals

Step 4 explicitly adds a fresh noise draw (future_noise) to each simulated value before collecting it. This makes the output a prediction interval (uncertainty for a new individual observation), not a confidence interval (uncertainty for the mean response or a parameter estimate).

Confidence interval target: E[y | x*] (the noiseless population regression value) or a parameter like β₁.
Prediction interval target: a new observation y* = E[y|x*] + noise.

Formula-wise, the difference is one term under the square root:

CI: ŷ* ± t·s·√(1/n + (x*−x̄)²/Sxx)
PI: ŷ* ± t·s·√(1 + 1/n + (x*−x̄)²/Sxx)

If confidence intervals are required: remove the future_noise addition in step 4 and take quantiles of the raw ŷ values (the fitted-line predictions across resampled datasets) instead.

Known Issue: x_train is re-randomized every iteration

In step 4, a new x_train (not just new noise) is generated on every Monte Carlo iteration. This means the interval marginalizes over randomness in the training design itself, not just parameter and noise uncertainty conditional on one fixed dataset. As a result, this pipeline quantifies unconditional/total uncertainty (as if the whole experiment — sample and all — were rerun), rather than the standard conditional interval (given one fixed, already-collected dataset).

If the conditional interval is required: generate x_train once, outside the loop, and only resample noise/y inside the loop.

Validation results

Coverage test confirms the intervals produced by this pipeline are well-calibrated for the unconditional prediction-interval target actually being simulated (empirical coverage tracks nominal coverage closely across 80/90/95/99%). This does not confirm calibration against the classical (conditional) prediction interval formula or against any confidence interval target.




Claude is AI and can make mistakes.
