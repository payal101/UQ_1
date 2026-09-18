# Bayesian Linear Regression + Prediction Intervals

This project studies **uncertainty quantification (UQ)** for linear regression by constructing and evaluating prediction intervals for a future observation \(y^*\).

The project starts with a **Monte Carlo OLS prediction-interval pipeline** and then extends it with closed-form OLS and Bayesian intervals, including intervals obtained using **Chernoff bounds**.

The main question is:

> **Given a prediction \(\hat y_*\), how can we construct an interval that contains a future observation \(y_*\) with a specified probability, and how can we verify that coverage experimentally?**

---

## 1. Problem Setup

We consider the linear model

$$
y = \beta_0+\beta_1x+\epsilon,
$$

where

$$
\epsilon\sim\mathcal N(0,\sigma^2).
$$

The true parameters used in the experiment are

$$
\beta_0=1,\qquad \beta_1=2,\qquad \sigma=1.
$$

For a new input \(x_*\), the goal is to construct a prediction interval

$$
C(x_*)=[L(x_*),U(x_*)]
$$

that contains the future observation \(y_*\) with approximately \(1-\alpha\) probability.

---

# 2. Methods

The project compares the following approaches.

| Method                | How the interval is constructed                                 | Main assumption                      |
| --------------------- | --------------------------------------------------------------- | ------------------------------------ |
| **Monte Carlo PI**    | Simulate many datasets, refit OLS, and take empirical quantiles | Simulation-based                     |
| **OLS Exact**         | Use the exact Gaussian prediction distribution                  | Gaussian noise                       |
| **OLS Chernoff**      | Use a Chernoff tail bound                                       | Sub-Gaussian error                   |
| **Bayesian Exact**    | Use the posterior predictive Gaussian distribution              | Gaussian prior + Gaussian likelihood |
| **Bayesian Chernoff** | Apply the Chernoff bound to the posterior predictive error      | Sub-Gaussian predictive error        |

The Monte Carlo method is simulation-based, while the other methods use closed-form expressions.

---

# 3. Monte Carlo Prediction Interval

The original notebook constructs the prediction interval by simulation.

For each Monte Carlo replicate:

1. Generate a new training dataset.
2. Fit the OLS model.
3. Generate a future observation \(y_*\).
4. Store the resulting prediction.
5. Repeat many times.

The empirical distribution of the simulated \(y_*\) values is then used to construct the interval.

For example, a \(95\%\) prediction interval is obtained from the empirical \(2.5\%\) and \(97.5\%\) quantiles.

This approach does not require deriving a closed-form prediction interval.

---

# 4. OLS Exact Prediction Interval

For OLS,

$$
\hat\beta=(X^TX)^{-1}X^Ty.
$$

For a new input \(x_*\), the predictive variance is

$$
v_{\mathrm{OLS}}
=
\sigma^2
\left(
1+x_*^T(X^TX)^{-1}x_*
\right).
$$

Under Gaussian noise, the prediction error is Gaussian.

Therefore, the exact prediction interval is

$$
\boxed{
\hat y_*
\pm
z_{1-\alpha/2}\sqrt{v_{\mathrm{OLS}}}
}
$$

where \(z_{1-\alpha/2}\) is the corresponding standard-normal quantile.

For example, at \(95\%\),

$$
z_{0.975}\approx1.96.
$$

---

# 5. Bayesian Linear Regression

We also fit a Bayesian linear regression model.

The prior is

$$
\beta\sim\mathcal N(m_0,S_0).
$$

Given the training data, the posterior distribution is

$$
\beta\mid D
\sim
\mathcal N(\mu_n,\Sigma_n).
$$

For a new input \(x_*\), the posterior predictive distribution is

$$
Y_*\mid x_*,D
\sim
\mathcal N(m_*,s_*^2),
$$

where

$$
m_*=x_*^T\mu_n
$$

and

$$
s_*^2
=
\sigma^2+x_*^T\Sigma_nx_*.
$$

The predictive variance contains two sources of uncertainty:

$$
\underbrace{\sigma^2}_{\text{observation noise}}
+
\underbrace{x_*^T\Sigma_nx_*}_{\text{parameter uncertainty}}.
$$

---

# 6. Bayesian Exact Prediction Interval

Since the posterior predictive distribution is Gaussian, the exact Bayesian prediction interval is

$$
\boxed{
m_*
\pm
z_{1-\alpha/2}s_*
}
$$

with

$$
s_*=\sqrt{\sigma^2+x_*^T\Sigma_nx_*}.
$$

Conditional on the observed data, this gives the Bayesian posterior predictive statement

$$
\Pr_{\mathrm{post}}
\left(
Y_*\in C_B(D,x_*)
\mid D
\right)
=
1-\alpha.
$$

This is a **Bayesian probability statement**. It should be distinguished from the frequentist question of whether repeated experiments achieve \(1-\alpha\) coverage.

---

# 7. Chernoff Bounds

Instead of using the exact Gaussian quantile, we can obtain an interval using a tail bound.

Let

$$
Z=Y_*-\hat y_*.
$$

Suppose \(Z\) is mean-zero and \(v\)-sub-Gaussian, meaning

$$
\mathbb E[e^{\lambda Z}]
\leq
e^{\lambda^2v/2}.
$$

Using Markov's inequality on \(e^{\lambda Z}\),

$$
\Pr(Z\geq t)
\leq
e^{-\lambda t}
\mathbb E[e^{\lambda Z}].
$$

Therefore,

$$
\Pr(Z\geq t)
\leq
\exp
\left(
-\lambda t+\frac{\lambda^2v}{2}
\right).
$$

Minimizing over \(\lambda\) gives

$$
\lambda^*=\frac{t}{v}.
$$

Substituting this value gives

$$
\Pr(Z\geq t)
\leq
e^{-t^2/(2v)}.
$$

Applying the same argument to the lower tail and using the union bound,

$$
\boxed{
\Pr(|Z|\geq t)
\leq
2e^{-t^2/(2v)}.
}
$$

To obtain an interval with failure probability at most \(\alpha\), set

$$
2e^{-t^2/(2v)}=\alpha.
$$

Solving for \(t\),

$$
\boxed{
t=
\sqrt{2v\log(2/\alpha)}.
}
$$

Therefore, the Chernoff prediction interval is

$$
\boxed{
\hat y_*
\pm
\sqrt{2v\log(2/\alpha)}.
}
$$

---

# 8. Why Use Chernoff?

When the full distribution is known, the exact Gaussian quantile is tighter.

For example, at \(95\%\),

$$
z_{0.975}\approx1.96,
$$

while the Chernoff factor is

$$
\sqrt{2\log(40)}
\approx2.72.
$$

Thus,

$$
1.96\sqrt v
<
2.72\sqrt v.
$$

The Chernoff interval is therefore wider.

The advantage is that the Chernoff argument does not require knowing the **exact Gaussian distribution**. A sub-Gaussian MGF bound is sufficient.

Thus the trade-off is:

$$
\boxed{
\text{weaker distributional assumption}
\quad\Longrightarrow\quad
\text{more conservative interval}.
}
$$

---

# 9. Applying Chernoff to OLS

For OLS, the prediction error is a linear combination of Gaussian noise terms, so it is Gaussian and therefore sub-Gaussian.

Using

$$
v_{\mathrm{OLS}}
=
\sigma^2
\left(
1+x_*^T(X^TX)^{-1}x_*
\right),
$$

the Chernoff interval becomes

$$
\boxed{
\hat y_*
\pm
\sqrt{
2v_{\mathrm{OLS}}\log(2/\alpha)
}.
}
$$

---

# 10. Applying Chernoff to Bayesian Prediction

For Bayesian linear regression,

$$
Y_*\mid x_*,D
\sim
\mathcal N(m_*,s_*^2).
$$

Therefore, using the same tail-bound argument with

$$
v=s_*^2
=
\sigma^2+x_*^T\Sigma_nx_*,
$$

we obtain

$$
\boxed{
m_*
\pm
\sqrt{
2s_*^2\log(2/\alpha)
}.
}
$$

---

# 11. Coverage Verification

Constructing an interval is only one part of the problem.

We also want to determine whether the interval actually achieves its nominal coverage when evaluated on fresh data.

For repeated experiments, define

$$
Z_i=
\mathbf 1
\{Y_i^*\in C_i\}.
$$

The empirical coverage is

$$
\boxed{
\hat p_N
=
\frac1N\sum_{i=1}^N Z_i.
}
$$

The corresponding frequentist predictive coverage is

$$
\boxed{
p_{\mathrm{cov}}
=
P^*(Y_*^*\in C(D,x_*)).
}
$$

We compare the empirical coverage \(\hat p_N\) with the nominal coverage \(1-\alpha\).

---

# 12. Concentration of the Coverage Estimate

Since the coverage indicators are Bernoulli random variables, a Chernoff--Hoeffding concentration inequality gives

$$
\boxed{
P^*
\left(
|\hat p_N-p_{\mathrm{cov}}|
\geq\epsilon
\right)
\leq
2e^{-2N\epsilon^2}.
}
$$

Equivalently, to obtain accuracy \(\epsilon\) with failure probability at most \(\delta\), it is sufficient to use

$$
\boxed{
N
\geq
\frac{\log(2/\delta)}
{2\epsilon^2}.
}
$$

Importantly, this concentration inequality does **not** prove that

$$
p_{\mathrm{cov}}=1-\alpha.
$$

Instead, it tells us that our empirical estimate \(\hat p_N\) is close to the true coverage \(p_{\mathrm{cov}}\) with high probability.

---

# 13. Experimental Setup

The experiments use:

* Training samples: \(n=50\)
* Test/coverage repetitions: \(N=2000\)
* True model:

  $$
  y=1+2x+\epsilon
  $$
* Noise:

  $$
  \epsilon\sim\mathcal N(0,1)
  $$
* \(x\) sampled uniformly from \([-3,3]\)
* Bayesian prior:

  $$
  \beta\sim\mathcal N(0,10I)
  $$

  which is relatively weak compared with the information from \(50\) observations.
* Nominal coverage levels:

  $$
  80\%,90\%,95\%,99\%.
  $$

For the comparison between the closed-form methods, the training design \(X\) is fixed and reused across coverage repetitions. Only the training noise and future observations are resampled.

This makes the experiment consistent with the conditional OLS and Bayesian formulas.

---

# 14. Visualizations

### Graph 1 — Monte Carlo Predictive Distribution

A histogram of the simulated \(y_*\) values at

$$
x_*=1.
$$

The true regression value is

$$
1+2(1)=3.
$$

The graph shows the simulated predictive distribution together with its empirical quantiles.

---

### Graph 2 — Monte Carlo Coverage

The empirical coverage of the Monte Carlo prediction interval is plotted against the nominal coverage.

The reference line

$$
y=x
$$

represents perfect calibration.

---

### Graph 3 — Comparison of All Closed-Form Methods

The following methods are compared:

* OLS Exact
* OLS Chernoff
* Bayesian Exact
* Bayesian Chernoff

The exact Gaussian intervals are expected to be close to the nominal coverage.

The Chernoff intervals can lie above the nominal coverage because the Chernoff inequality is an upper bound on the tail probability and is therefore generally conservative.

---

# 15. OLS vs Bayesian Regression

In the current experiment, the OLS and Bayesian results are expected to be similar because the prior

$$
\beta\sim\mathcal N(0,10I)
$$

is relatively weak compared with \(n=50\) observations.

To make the Bayesian effect more visible, we can:

1. Reduce the prior scale, making the prior more informative.
2. Reduce the number of training observations.

Then the posterior will be influenced more strongly by the prior, and the Bayesian predictive intervals can differ more noticeably from the OLS intervals.

---

# 16. Main Takeaway

The project illustrates the distinction between **constructing uncertainty intervals** and **verifying their coverage**.

$$
\boxed{
\text{Model}
\rightarrow
\text{Predictive distribution}
\rightarrow
\text{Prediction interval}
\rightarrow
\text{Coverage verification}
}
$$

Bayesian inference provides a posterior predictive distribution.

Exact Gaussian methods use the known distribution directly.

Chernoff bounds replace the exact tail probability with a provable upper bound, allowing intervals under weaker sub-Gaussian assumptions at the cost of conservativeness.

Finally, repeated sampling allows us to empirically evaluate the frequentist coverage of the resulting intervals, while concentration inequalities quantify the accuracy of that empirical coverage estimate.
