# README — Monte Carlo Prediction Intervals

## What this notebook does

Simulates the process of fitting a linear regression model over and over (5000 times), builds a prediction interval from those simulations, then checks if that interval is actually trustworthy by repeating the whole thing 1000 more times.

**True model:** y = 1 + 2x + noise

---

## The steps, in plain terms

1. **Make fake data** — generate 50 (x, y) points from the true model.
2. **Fit a line** — run ordinary least squares on that fake data.
3. **Predict** — use the fitted line to predict y at x = 1.
4. **Add noise** — add one more random noise value on top, to simulate what an actual new observation would look like.
5. **Repeat 5000 times** — steps 1–4, each time with brand new fake data. Collect all 5000 results.
6. **Build the interval** — take the middle 95% of those 5000 values → that's the interval.
7. **Check it works** — repeat everything (steps 1–6) 1000 times, and each time check if a real new data point actually falls inside the interval. Do this at 80%, 90%, 95%, and 99% confidence levels.

**Result: it works.** The intervals contain the true value about as often as they're supposed to.

---

## Two things to know before using this

### 1. This is a prediction interval, not a confidence interval

- **Prediction interval** = range for one new data point (includes noise)
- **Confidence interval** = range for the true underlying line/average (no noise added)

Step 4 adds noise — that's what makes this a prediction interval. If you need a confidence interval instead, skip step 4 and just use the raw predictions from step 3.

### 2. The interval is a bit broader than the "standard" one

Every time this generates fake data (step 1), it randomizes everything — including the x-values, not just the noise. The usual textbook interval assumes you already have your data and only the noise is random. Because this version also randomizes the x's, it's measuring "what if I reran the whole experiment from scratch" rather than "what if I just got a new data point using the data I already have." Both are valid things to measure, but they're not the same number, and this one will come out a bit wider.

**Fix if you want the standard version:** generate the fake data once, outside the loop, and only re-randomize the noise inside the loop.
