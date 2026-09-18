# Bayesian Linear Regression + Prediction Intervals

This project studies **uncertainty in linear regression predictions**.

The main goal is to construct prediction intervals for a future observation \(y^*\) and check whether those intervals achieve their intended coverage.

The notebook compares **simulation-based, exact Gaussian, and Chernoff-bound approaches**.

---

## 1. Basic Setup

We use the linear model

$$
y = \beta_0 + \beta_1x + \epsilon
$$

with

$$
\beta_0=1,\qquad \beta_1=2,\qquad \epsilon\sim N(0,1).
$$

For a new input \(x^*\), we want an interval

$$
[L,U]
$$

that contains the future value \(y^*\) with probability approximately \(1-\alpha\).

For example, for a 95% interval,

$$
1-\alpha=0.95.
$$

---

## 2. Methods Compared

We use five approaches:

| Method                | Idea                                                             |
| --------------------- | ---------------------------------------------------------------- |
| **Monte Carlo OLS**   | Simulate many datasets and use the empirical quantiles           |
| **OLS Exact**         | Use the exact Gaussian prediction interval                       |
| **OLS Chernoff**      | Use a Chernoff bound instead of the Gaussian quantile            |
| **Bayesian Exact**    | Use the Bayesian posterior predictive distribution               |
| **Bayesian Chernoff** | Apply the Chernoff bound to the Bayesian predictive distribution |

The main comparison in the final experiment is between the **four closed-form methods**.

---

## 3. Monte Carlo Prediction Interval

The original notebook uses simulation.

We repeatedly:

1. Generate training data.
2. Fit OLS.
3. Generate a future observation.
4. Store the prediction.
5. Repeat many times.

The resulting simulated distribution is used to obtain prediction intervals from empirical quantiles.

This gives us a baseline to compare against the closed-form methods.

---

## 4. Bayesian Prediction

For Bayesian linear regression, we place a Gaussian prior on the regression parameters.

After observing the data, we obtain a posterior distribution for the parameters.

This gives a **posterior predictive distribution** for a new observation:

$$
Y^*|x^*,D
\sim
N(m^*,s^{*2})
$$

where

$$
m^*=x^{*T}\mu_n
$$

and

$$
s^{*2}
=
\sigma^2+x^{*T}\Sigma_nx^*.
$$

The predictive uncertainty contains:

* uncertainty in the regression parameters;
* observation noise.

The exact Bayesian prediction interval is obtained from the Gaussian quantile:

$$
m^*
\pm
z_{1-\alpha/2}s^*.
$$

---

## 5. Chernoff Prediction Interval

Instead of using the exact Gaussian quantile, we can use a **Chernoff bound**.

For a mean-zero \(v\)-sub-Gaussian random variable \(Z\),

$$
P(|Z|\geq t)
\leq
2\exp\left(-\frac{t^2}{2v}\right).
$$

Setting the right-hand side equal to \(\alpha\) gives

$$
t=
\sqrt{2v\log(2/\alpha)}.
$$

Therefore, the Chernoff interval is

$$
\boxed{
\text{prediction}
\pm
\sqrt{2v\log(2/\alpha)}
}
$$

The important idea is:

> **Chernoff does not need the full Gaussian distribution. A sub-Gaussian bound is enough.**

The trade-off is that the resulting interval is usually wider.

---

## 6. Checking Coverage

After constructing an interval, we need to check whether it actually covers future observations at the expected rate.

For each experiment, define

$$
Z_i =
\mathbf 1\{y_i^*\text{ lies inside the interval}\}.
$$

The empirical coverage is

$$
\hat p_N
=
\frac{1}{N}\sum_{i=1}^N Z_i.
$$

For example, if we construct a 95% interval and 1910 out of 2000 future observations fall inside it,

$$
\hat p_N=\frac{1910}{2000}=0.955.
$$

We then compare empirical coverage with the nominal coverage.

---

## 7. Chernoff–Hoeffding Bound for Coverage

The coverage indicators are Bernoulli random variables.

Therefore,

$$
P(|\hat p_N-p_{\mathrm{cov}}|\geq\epsilon)
\leq
2e^{-2N\epsilon^2}.
$$

This tells us how accurately we can estimate the true coverage using \(N\) experiments.

**Important:** this bound does not prove that the interval has 95% coverage.

It only tells us that the empirical coverage is close to the true coverage with high probability.

---

## 8. Experiments

We use:

* \(n=50\) training samples
* \(N=2000\) coverage experiments
* True model:

  $$
  y=1+2x+\epsilon
  $$
* \(\epsilon\sim N(0,1)\)
* Nominal coverage levels:

  $$
  80\%,90\%,95\%,99\%.
  $$

For the closed-form comparison, the training design is fixed and the noise/future observations are resampled.

---

## 9. Plots

### Plot 1 — Monte Carlo Predictive Distribution

Shows the simulated predictive distribution for a fixed \(x^*\).

The true prediction at \(x^*=1\) is

$$
1+2(1)=3.
$$

The plot shows that the simulated distribution is centered close to this value.

### Plot 2 — Monte Carlo Coverage

Plots empirical coverage against nominal coverage.

The diagonal line

$$
y=x
$$

represents perfect agreement.

### Plot 3 — Closed-Form Comparison

Compares:

* OLS Exact
* OLS Chernoff
* Bayesian Exact
* Bayesian Chernoff

The exact Gaussian methods should be close to the nominal coverage.

The Chernoff methods are expected to be more conservative because the bound is looser than the exact Gaussian tail probability.

---

## 10. Main Takeaway

The notebook demonstrates the following pipeline:

$$
\boxed{
\text{Model}
\rightarrow
\text{Predictive uncertainty}
\rightarrow
\text{Prediction interval}
\rightarrow
\text{Coverage check}
}
$$

The main comparison is between **exact distribution-based intervals** and **Chernoff-bound intervals**.

The exact Gaussian approach gives tighter intervals when the Gaussian assumption is valid.

Chernoff bounds give a more general way to construct intervals under weaker sub-Gaussian assumptions, but can produce wider, more conservative intervals.

Finally, repeated experiments allow us to check how the empirical coverage compares with the desired nominal coverage.
