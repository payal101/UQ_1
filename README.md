# Bayesian Linear Regression + Prediction Intervals

This project studies **uncertainty in linear regression predictions**.

The main goal is to construct prediction intervals for a future observation \(y^*\) and evaluate whether those intervals achieve their intended coverage.

The notebook compares **simulation-based, exact Gaussian, and Chernoff-bound approaches**.

---

## 1. Basic Setup

We consider the linear model

$$
y = \beta_0 + \beta_1 x + \epsilon,
$$

where

$$
\beta_0 = 1,
\qquad
\beta_1 = 2,
\qquad
\epsilon \sim \mathcal N(0,1).
$$

For a new input \(x^*\), we want to construct an interval

$$
[L,U]
$$

that contains the future observation \(y^*\) with probability approximately

$$
1-\alpha.
$$

For example, a 95% prediction interval corresponds to

$$
1-\alpha = 0.95.
$$

---

## 2. Methods Compared

The notebook considers five approaches:

| Method                | Idea                                                                               |
| --------------------- | ---------------------------------------------------------------------------------- |
| **Monte Carlo OLS**   | Simulate many datasets and estimate prediction intervals using empirical quantiles |
| **OLS Exact**         | Use the exact Gaussian prediction interval                                         |
| **OLS Chernoff**      | Replace the Gaussian quantile with a Chernoff bound                                |
| **Bayesian Exact**    | Use the Bayesian posterior predictive distribution                                 |
| **Bayesian Chernoff** | Apply a Chernoff bound to the Bayesian predictive distribution                     |

The main closed-form comparison is between the four OLS/Bayesian exact and Chernoff methods.

---

## 3. Monte Carlo Prediction Interval

The notebook first constructs prediction intervals using simulation.

For each experiment, we:

1. Generate a training dataset.
2. Fit the regression model.
3. Generate a future observation.
4. Record the prediction.
5. Repeat the experiment many times.

The empirical distribution of the resulting predictions is then used to construct prediction intervals using empirical quantiles.

This provides a simulation-based baseline for comparison with the analytical methods.

---

## 4. Bayesian Linear Regression

For Bayesian linear regression, we place a Gaussian prior on the regression parameters.

After observing the training data, we obtain a posterior distribution over the parameters.

For a new input \(x^*\), this posterior induces a **posterior predictive distribution**

$$
Y^* \mid x^*,D
\sim
\mathcal N(m^*,s^{*2}),
$$

where

$$
m^* = x^{*T}\mu_n
$$

and

$$
s^{*2}
=
\sigma^2 + x^{*T}\Sigma_n x^*.
$$

The predictive variance contains two sources of uncertainty:

* **Parameter uncertainty:** uncertainty about the regression coefficients.
* **Observation noise:** the irreducible noise in the future observation.

The exact Bayesian prediction interval is therefore

$$
\boxed{
m^*
\pm
z_{1-\alpha/2}s^*
}
$$

where \(z_{1-\alpha/2}\) is the corresponding standard Gaussian quantile.

---

## 5. Chernoff Prediction Interval

Instead of using the exact Gaussian quantile, we can construct an interval using a **Chernoff bound**.

Suppose \(Z\) is a mean-zero \(v\)-sub-Gaussian random variable. Then

$$
P(|Z|\geq t)
\leq
2\exp\left(
-\frac{t^2}{2v}
\right).
$$

To obtain a confidence level \(1-\alpha\), set

$$
2\exp\left(
-\frac{t^2}{2v}
\right)
=
\alpha.
$$

Solving for \(t\),

$$
t
=
\sqrt{2v\log\left(\frac{2}{\alpha}\right)}.
$$

Therefore, a Chernoff prediction interval has the form

$$
\boxed{
\text{prediction}
\pm
\sqrt{
2v\log\left(\frac{2}{\alpha}\right)
}
}
$$

The key idea is:

> **Chernoff bounds do not require the full distribution. A suitable tail or sub-Gaussian bound is sufficient.**

The trade-off is that the resulting interval can be wider than the exact Gaussian interval.

---

## 6. Coverage

Constructing an interval is only part of the problem. We also need to evaluate whether it actually achieves the desired coverage.

For each experiment, define the indicator

$$
Z_i
=
\mathbf 1
\left\{
y_i^* \in [L_i,U_i]
\right\}.
$$

The empirical coverage over \(N\) experiments is

$$
\hat p_N
=
\frac{1}{N}
\sum_{i=1}^{N} Z_i.
$$

For example, if a nominal 95% interval contains 1910 out of 2000 future observations, then

$$
\hat p_N
=
\frac{1910}{2000}
=
0.955.
$$

We compare this empirical coverage with the desired nominal coverage.

---

## 7. Chernoff–Hoeffding Bound for Coverage Estimation

The coverage indicators \(Z_i\) are Bernoulli random variables.

Therefore, Hoeffding's inequality gives

$$
P
\left(
|\hat p_N-p_{\mathrm{cov}}|
\geq \epsilon
\right)
\leq
2e^{-2N\epsilon^2}.
$$

This provides a probabilistic guarantee on how close the empirical coverage \(\hat p_N\) is to the true coverage \(p_{\mathrm{cov}}\).

### Important distinction

This bound **does not prove that the prediction interval has 95% coverage**.

Instead, it tells us how reliably we can estimate the interval's actual coverage using a finite number of experiments.

Thus there are two separate uses of concentration bounds in the experiment:

$$
\boxed{
\text{Chernoff bound for interval construction}
}
$$

and

$$
\boxed{
\text{Hoeffding bound for estimating coverage}
}
$$

---

## 8. Experimental Setup

The coverage experiment uses:

* Training samples:

  $$
  n=50
  $$

* Number of coverage experiments:

  $$
  N=2000
  $$

* True regression model:

  $$
  y=1+2x+\epsilon
  $$

* Noise:

  $$
  \epsilon\sim\mathcal N(0,1)
  $$

* Nominal coverage levels:

  $$
  80\%,\quad90\%,\quad95\%,\quad99\%.
  $$

For the closed-form comparison, the training design is fixed while the training noise and future observations are resampled across experiments.

---

## 9. Plots

### Plot 1 — Monte Carlo Predictive Distribution

This plot shows the simulated predictive distribution for a fixed \(x^*\).

For example, at

$$
x^*=1,
$$

the mean of the true regression function is

$$
\beta_0+\beta_1x^*
=
1+2(1)
=
3.
$$

The simulated predictive distribution should therefore be centered close to \(3\).

---

### Plot 2 — Monte Carlo Coverage

This plot compares empirical coverage with nominal coverage.

The diagonal line

$$
y=x
$$

represents perfect agreement between the desired and observed coverage.

Points close to the diagonal indicate that the empirical coverage is close to the nominal coverage.

---

### Plot 3 — Closed-Form Coverage Comparison

This plot compares:

* OLS Exact
* OLS Chernoff
* Bayesian Exact
* Bayesian Chernoff

The exact Gaussian methods should be close to their nominal coverage when their assumptions are satisfied.

The Chernoff methods may produce more conservative intervals because a concentration bound generally provides a looser tail guarantee than the exact Gaussian quantile.

---

## 10. Overall Pipeline

The experiment follows the pipeline

$$
\boxed{
\text{Data}
\rightarrow
\text{Regression Model}
\rightarrow
\text{Predictive Distribution}
\rightarrow
\text{Prediction Interval}
\rightarrow
\text{Coverage Experiment}
}
$$

The central comparison is between **distribution-specific uncertainty quantification** and **bound-based uncertainty quantification**.

The exact Gaussian approach uses knowledge of the full predictive distribution to construct a tighter interval.

The Chernoff approach uses a tail bound and therefore requires less distributional information, at the cost of potentially wider intervals.

Finally, repeated experiments allow us to empirically evaluate whether the constructed intervals achieve their intended coverage.
