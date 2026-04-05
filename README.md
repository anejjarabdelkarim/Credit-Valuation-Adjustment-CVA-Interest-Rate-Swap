# Credit Valuation Adjustment (CVA) — Interest Rate Swap

A simulation-based framework for estimating the **Credit Valuation Adjustment (CVA)** of a plain vanilla interest rate swap, using real market data and progressively sophisticated credit risk models.

---

## 📌 Overview

CVA represents the market value of counterparty default risk embedded in a derivative contract. This project builds a full CVA pricing pipeline from scratch, covering yield curve construction, Monte Carlo simulation of interest rate paths, swap valuation, and credit risk modeling under several assumptions — including **Wrong-Way Risk (WWR)**.

---

## 📂 Project Structure
```
├── data/                        # Raw FRED swap rate data
├── CVA_notebook.ipynb           # Main notebook
└── README.md
```

---

## 🔧 Methodology

### 1. Market Data
- Historical swap rates (1Y, 2Y, 5Y, 10Y, 30Y) sourced from **FRED**
- Rates merged and aligned by observation date

### 2. Yield Curve Construction
- **Cubic spline interpolation** over quoted maturities
- Discount factors derived via exponential discounting: $D(t) = e^{-r(t) \cdot t}$

### 3. Monte Carlo Simulation
- Short rate modeled as **Brownian motion with drift**: $r_{t+\Delta t} = r_t + \mu \Delta t + \sigma \sqrt{\Delta t} \varepsilon_t$
- Parameters $\mu$ and $\sigma$ estimated from historical 1Y swap rates
- 1,000 paths over a 10-year horizon with quarterly time steps

### 4. IRS Valuation
- Pay-fixed, receive-floating plain vanilla swap
- Fixed leg: $V_{\text{fix}}(t) = K \sum_{t_i > t} \Delta t \cdot D(t, t_i)$
- Floating leg: $V_{\text{float}}(t) = D(t, t_{\text{next}}) - D(t, T)$
- Fair fixed rate $K^*$ calibrated so that $V(0) = 0$

### 5. Expected Exposure & PFE
- $\text{EE}(t) = \mathbb{E}[\max(V(t), 0)]$
- $PFE_{\alpha}(t) = \text{Quantile}_{\alpha}[\max(V(t), 0)]$

### 6. CVA Models

| Model | Default Probability |
|-------|-------------------|
| Flat PD | $PD(t_{i-1}, t_i) \approx PD \cdot \Delta t$ |
| Constant hazard rate | $S(t) = e^{-\lambda t}$ |
| Time-dependent hazard rate | $S(t) = \exp\left(-\lambda_0 t - \frac{\alpha}{2}t^2\right)$ |
| Wrong-Way Risk (WWR) | $\lambda(t) = \lambda_0 \cdot \exp(\gamma \cdot \bar{r}(t))$ |

CVA formula:
$$\text{CVA} = (1-R) \sum_{i=1}^{n} \text{EE}(t_i)\cdot \left(S(t_{i-1}) - S(t_i)\right)$$

## 📊 Key Results

| Configuration | Model | CVA (€) |
|---------------|-------|---------|
| Off-market ($K = 2\%$) | Flat PD | 26,835.92 |
| Off-market ($K = 2\%$) | Exponential survival | 24,812.29 |
| Off-market ($K = 2\%$) | Time-dependent hazard | 29,243.85 |
| Off-market ($K = 2\%$) | WWR |  $31{,}000.58$ |
| Fair rate ($K^*$) | Flat PD | 1,366.99 |
| Fair rate ($K^*$) | Exponential survival | 1,310.66 |
| Fair rate ($K^*$) | Time-dependent hazard | 1,740.79 |
| Fair rate ($K^*$) | WWR | $1{,}646.21$ |

---

## 💡 Key Findings

- **Calibration dominates model choice** — an off-market swap inflates CVA by ~18x regardless of the default model used
- **Wrong-Way Risk** adds a measurable CVA premium by linking hazard rates to simulated interest rates, capturing the adverse scenario where the counterparty is most likely to default when our exposure is highest
- **Time-dependent hazard rates** consistently produce higher CVA than constant models, reflecting greater long-term default uncertainty
- **PFE** provides a conservative, regulatory-grade bound on worst-case exposure at the 95% confidence level

---

## 🛠️ Requirements
```bash
pip install numpy pandas matplotlib scipy fredapi
```

---

## 📦 Dependencies

| Library | Usage |
|---------|-------|
| `numpy` | Numerical computations |
| `pandas` | Data manipulation |
| `matplotlib` | Visualization |
| `scipy` | Cubic spline interpolation |
| `fredapi` | FRED data extraction |

---

## 📖 References

- Gregory, J. (2015). *The xVA Challenge: Counterparty Credit Risk, Funding, Collateral and Capital*. Wiley.
- Basel Committee on Banking Supervision (2011). *Basel III: A global regulatory framework for more resilient banks*.
- Federal Reserve Economic Data (FRED) — [https://fred.stlouisfed.org](https://fred.stlouisfed.org)

---

## 👤 Author

Project developed as part of a quantitative finance study on counterparty credit risk and xVA modeling.
