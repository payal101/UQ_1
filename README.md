# Part 2 — Bayesian Linear Regression + Chernoff-Bound Intervals

This extends `uq1.ipynb` (your Monte Carlo OLS prediction-interval pipeline)
with two more ways to build an interval, plus a proof for one of them, plus a
coverage comparison across all four.

## What was added

Your original notebook builds prediction intervals one way: **simulate**
many resampled datasets, refit OLS each time, and take quantiles of the
simulated `y*` values (Monte Carlo). The new cells add three more estimators
for the *same kind of interval* (a range that should contain a new
observation `y*` with some probability), so you can compare them head to head:

| Method | How the interval is built | Assumes |
|---|---|---|
| **Monte Carlo PI** (yours) | Simulate thousands of resampled datasets, take empirical quantiles | Nothing closed-form — just simulation |
| **OLS exact** | Closed-form Gaussian quantile: `y_hat ± z · sqrt(v)` | Noise is *exactly* Gaussian |
| **OLS Chernoff** | Closed-form Chernoff bound: `y_hat ± sqrt(2v·log(2/δ))` | Noise is only *sub-Gaussian* (weaker) |
| **Bayes exact** | Posterior predictive Gaussian quantile | Gaussian prior + Gaussian likelihood, noise exactly Gaussian |
| **Bayes Chernoff** | Chernoff bound on the same posterior predictive | Sub-Gaussian only |

`v` is the predictive variance — for OLS it's the classical
`σ²(1 + x*ᵀ(XᵀX)⁻¹x*)` from your README; for Bayes it's
`x*ᵀΣₙx* + σ²` from the posterior covariance `Σₙ`.

## The proof (markdown cell in the notebook)

The Chernoff bound is derived from scratch via the moment-generating-function
(MGF) technique — not just asserted:

1. **Chernoff's technique**: for any r.v. `Z`, Markov's inequality on `e^{λZ}`
   gives `P(Z≥t) ≤ E[e^{λZ}]·e^{-λt}` for every `λ>0`; minimizing over `λ`
   gives the tightest such bound.
2. **Applied to `Z ~ N(0,v)`**: the MGF is `exp(λ²v/2)`, minimizing over `λ`
   gives `λ* = t/v`, yielding `P(Z≥t) ≤ exp(-t²/2v)`, and by symmetry +
   union bound, `P(|Z|≥t) ≤ 2exp(-t²/2v)`. Solving for `t` at level `δ`
   gives the interval half-width `t(δ) = sqrt(2v·log(2/δ))`.
3. **Key point**: step 2 only used the MGF of a Gaussian, not any other
   Gaussian-specific property — so the same bound holds for *any*
   mean-zero, `v`-sub-Gaussian `Z` (bounded noise, etc.), not just exactly
   Gaussian noise. That's the whole reason to use it: it buys robustness to
   the exact-normality assumption, at the cost of a wider interval.
4. Then it's applied to the actual prediction error in both the OLS case
   (`ŷ - β` is Gaussian because it's a linear map of Gaussian noise) and the
   Bayesian posterior predictive.

## Graph 1 — your original: Monte Carlo Predictive Distribution (histogram)

This one is unchanged from your notebook — a histogram of the 5000 simulated
`y*` values at `x* = 1`, with dashed lines at the 2.5%, median, and 97.5%
quantiles. It's just a visual check that the simulated predictive
distribution looks Gaussian-ish and centered near the true value
`β0+β1·x* = 1+2 = 3`, which it does (median ≈ 2.99).

## Graph 2 — your original: Coverage Calibration (Monte Carlo only)

Also unchanged: empirical coverage of the Monte Carlo interval vs. nominal
confidence, at 80/90/95/99%, against the `y=x` "perfect calibration" line.
Your Monte Carlo method tracks it closely (e.g. 0.958 empirical vs. 0.95
nominal) — confirming it's well-calibrated for the *unconditional* target it
actually simulates (fresh `x_train` every replicate).

## Graph 3 — new: Coverage Calibration, all four methods

Same idea as Graph 2, but now plots four lines against the same `y=x`
reference, and — importantly — under a **different, fixed-design (conditional)
setup**: `x_train` is drawn once and reused across all 2000 coverage-test
replicates, only `y_train` (noise) and the future `y*` are resampled each
time. This is necessary because the OLS/Bayes closed forms are conditional
formulas (`Σₙ`, `(XᵀX)⁻¹` computed on a fixed design) — testing them against
your unconditional Monte Carlo loop would be a mismatched comparison, not
just a stricter one.

Reading the graph:
- **OLS exact** and **Bayes exact** sit almost exactly on the `y=x` line —
  they hit their nominal coverage (e.g. ~0.95 empirical at 0.95 nominal).
  They're nearly identical to each other because the prior used
  (`prior_scale=10`, i.e. weak) barely pulls the Bayesian posterior away from
  the OLS estimate given `n=50` data points.
- **OLS Chernoff** and **Bayes Chernoff** sit clearly *above* the `y=x` line
  at every nominal level (e.g. ~0.99 empirical at 0.95 nominal) — this is the
  proof made visible: the Chernoff bound is a valid but conservative
  (over-covering) interval, wider than the exact Gaussian quantile, because
  it only assumes sub-Gaussianity rather than exact normality.
- The gap between the "exact" pair and the "Chernoff" pair shows the price of
  that robustness — roughly the factor `sqrt(2log(2/δ))` vs. the tighter
  `z_{δ/2}`.

## Where the two setups would actually diverge

Right now Bayes and OLS nearly coincide because the prior is weak relative to
the data. To make the Bayesian side matter — and see the posterior actually
pull away from OLS — shrink `prior_scale` in `bayes_fit` (a tighter,
more informative prior), or reduce `n` in `x_train_fixed` so the likelihood
has less data to dominate with.
