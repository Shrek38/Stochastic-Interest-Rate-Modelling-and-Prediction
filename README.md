# Stochastic Interest Rate Modelling and Yield Curve Prediction

This project implements and extends the **Cox-Ingersoll-Ross (CIR)** model to reconstruct full yield curves from a single observable input — the 3-Month short rate. Built as part of the Finance Club, IIT Roorkee Open Projects 2026.

---

## What this project does

Interest rates don't sit still. They move in complex, seemingly random ways that make forecasting them genuinely hard. This project tackles that problem using stochastic calculus, specifically by implementing the CIR model, calibrating it on real historical yield data and then asking:

> *Given only today's 3-Month rate, can we reconstruct what the entire yield curve looks like?*

The answer turns out to be yes, with the right model and calibration strategy.

---

## Models Implemented

### 1. Base CIR Model
The CIR model describes how the short rate $r_t$ evolves over time:

$$dr_t = \kappa(\theta - r_t)dt + \sigma\sqrt{r_t}\,dW_t$$

- $\kappa$ controls how fast rates snap back to their long-run average after a shock
- $\theta$ is the long-run mean the rate gravitates toward
- $\sigma$ governs the size of random fluctuations

The square-root diffusion term ensures rates never go negative (provided the Feller condition $2\kappa\theta \geq \sigma^2$ holds), which is a key advantage over simpler models like Vasicek.

Parameters are calibrated using **Cross-Sectional Least Squares** via Differential Evolution — a global optimizer chosen specifically because the loss surface is non-convex and gradient-based methods tend to get stuck in local minima.

### 2. Two-Factor CIR Model (Extension)
The base model has one big limitation: it forces the entire yield curve to be driven by a single number. In reality, the curve shifts, steepens, flattens and inverts due to multiple independent forces.

The two-factor extension splits the short rate into two latent components:

$$r_t = x_t + y_t$$

where $x_t$ captures the **level** of rates and $y_t$ captures the **slope**. Since these factors aren't directly observable, a **Kalman Filter** is used to estimate them dynamically from the observed 3M rate over time.

---

## The Prediction Challenge

At test time, the model is only allowed to see the **3-Month yield** for each day. From that single number, it reconstructs the yields at 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, and 30Y maturities — and compares against the actual held-out values.

The target metric is an out-of-sample $R^2 > 0.85$.

---

## Results

| Model | Overall R² | Pass / Fail |
|---|---|---|
| Base CIR | 0.8927 | PASS |
| Two-Factor CIR | 0.9180 | PASS |

---

## Project Structure

```
stochastic-interest-rate-modelling/
│
├── yield_curve_prediction.ipynb   # Main Colab notebook (run top to bottom)
├── README.md
```

Data files (`train_data.csv`, `test_data.csv`, `test_data_3M.csv`) are loaded separately via the Finance Club's provided Google Drive link.

---

## Key concepts covered

- Stochastic differential equations and Itô calculus
- Zero-coupon bond pricing under the CIR framework
- Cross-sectional calibration vs MLE vs GMM — and why it matters
- Kalman Filter for latent state estimation in a linear state-space model
- Feller condition and what happens when it breaks
- Mean reversion half-life: how long interest rate shocks actually last
- Systematic model limitations at long maturities (5Y–30Y)

---

## Limitations (honest ones)

- Both models struggle at the long end of the curve (5Y–30Y). Long-maturity yields are driven by inflation expectations and term premiums — things a short-rate model fundamentally cannot capture.
- The Kalman Filter assumes Gaussian noise, which is an approximation. An Unscented Kalman Filter would be more theoretically correct.
- Parameters are calibrated once on a fixed training window. In a live setting, you'd recalibrate on a rolling basis as the rate environment shifts.
- Neither model accounts for the market price of risk, which means longer-duration bonds would be systematically mispriced in a real trading context.

---

## Tech Stack

- Python 3
- `numpy`, `pandas`, `scipy`, `scikit-learn`
- `matplotlib` for visualizations
- Google Colab

---
