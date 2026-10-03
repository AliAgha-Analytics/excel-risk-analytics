# Exposure Risk Analytics

An Excel-based trading portfolio risk analysis project covering **Value at Risk (VaR), Conditional VaR (CVaR), EWMA volatility, stress testing, correlation analysis, volatility-based sensitivity analysis and scenario analysis**.

The workbook was designed as a practical demonstration of how portfolio risk can be evaluated across multiple asset classes and under different market conditions.

> **Note:** This is a demonstration/portfolio project and is not intended to represent a production risk-management system.

---

## Project Overview

The portfolio contains six instruments:

* EURUSD
* XAUUSD (Gold)
* XAGUSD (Silver)
* USDJPY
* S&P 500
* NASDAQ

The workbook contains five main sections:

1. **VaR** – Historical, Parametric and Monte Carlo VaR/CVaR, holding-period sensitivity and rolling risk
2. **EWMA Volatility** – Time-varying volatility estimation using Exponentially Weighted Moving Average methodology
3. **Stress** – Multi-asset stress-testing scenarios
4. **Correlation** – Correlation analysis across instruments
5. **Scenarios** – ATR-based scenario analysis with long/short exposures and interactive case selection

---

# 1. VaR

The **VaR** sheet contains **250 trading days of historical returns** for EURUSD, XAUUSD, XAGUSD, USDJPY, S&P 500 and NASDAQ.

Portfolio returns are calculated using the assumed portfolio weights.

A small supporting table contains the **portfolio weight and notional value** for each instrument.

### Exposure Assumption

For simplicity, the initial portfolio assumes that all positions are **long exposures**.

For a short position, the corresponding return would need to be multiplied by **-1** before calculating the portfolio return.

---

## Historical Simulation VaR

Historical Simulation VaR is calculated directly from the historical portfolio-return distribution.

For the 95% confidence level:

1. Calculate the daily portfolio return.
2. Find the **5th percentile** of portfolio returns.
3. Treat this percentile as the daily VaR return.
4. Multiply the VaR return by the total portfolio notional to obtain the VaR in USD.

The same methodology is applied at the **99% confidence level**, using the 1st percentile.

### Conditional VaR / Expected Shortfall

CVaR (also referred to as Expected Shortfall) measures the **average loss beyond the VaR threshold**.

For the historical approach, CVaR is calculated as the average of all portfolio returns that fall below the relevant VaR percentile.

Both **95% and 99%** confidence levels are included in the analysis.

---

## Parametric (Variance-Covariance) VaR

The Parametric VaR approach assumes portfolio returns can be described using a normal distribution.

The workbook calculates:

* Portfolio mean return
* Portfolio standard deviation
* 95% VaR
* 99% VaR
* 95% CVaR
* 99% CVaR

The VaR calculation is based on the portfolio mean and standard deviation:

```text
VaR = Mean Return − Z × Standard Deviation
```

where:

* **Z ≈ 1.645** for 95% confidence
* **Z ≈ 2.326** for 99% confidence

The resulting return-based VaR is then converted into a dollar amount using the total portfolio notional.

### Parametric CVaR

CVaR is calculated using the normal-distribution Expected Shortfall formula:

```text
CVaR = Mean Return − [φ(Z) / (1 − Confidence Level)] × Standard Deviation
```

where `φ(Z)` represents the standard normal probability density function.

---

## Monte Carlo VaR

A simplified Monte Carlo approach is also included.

For demonstration purposes, **250 simulated portfolio-return observations** are generated using the portfolio's estimated mean and standard deviation.

The simulated returns are then used to calculate VaR and CVaR using the same percentile-based methodology as the Historical Simulation approach.

A production implementation would generally use a significantly larger number of simulations, such as **10,000 or more**, depending on the model design and computational requirements.

---

## Holding Period Sensitivity

The workbook includes VaR sensitivity across different holding periods:

* 1 day
* 5 days
* 14 days
* 22 days

For the sensitivity analysis, VaR is scaled using the square-root-of-time relationship:

```text
VaR(T) = VaR(1) × √T
```

This provides an estimate of how risk changes as the assumed holding period increases.

A line chart compares:

* Historical VaR 95%
* Historical CVaR 95%
* Historical VaR 99%
* Historical CVaR 99%

across the four holding periods.

---

## Rolling 60-Day VaR & CVaR

The sheet also includes a **60-day rolling Historical VaR and CVaR** calculation.

Rather than using the full historical dataset for every observation, the calculation uses only the most recent **60 trading days** at each point.

This helps illustrate how measured portfolio tail risk can change over time.

---

# 2. EWMA Volatility

The **EWMA Volatility** sheet estimates time-varying portfolio volatility using the **Exponentially Weighted Moving Average (EWMA)** methodology.

Unlike a simple historical standard deviation, which gives equal importance to observations within the selected sample, EWMA assigns **greater weight to more recent portfolio returns**.

This makes EWMA particularly useful for monitoring how portfolio risk changes when market conditions become more or less volatile.

The analysis uses two different decay factors:

* **λ = 0.94**
* **λ = 0.97**

Using two values allows the workbook to demonstrate how the responsiveness of the volatility estimate changes depending on the chosen decay factor.

---

## EWMA Variance

The EWMA approach estimates variance recursively using the following formula:

```text
Variance(t) = (1 − λ) × Return(t)² + λ × Variance(t − 1)
```

where:

* `λ` = decay factor
* `Return(t)` = current portfolio return
* `Variance(t − 1)` = previous day's EWMA variance

For the **0.94** specification:

```text
Variance(t) = 0.06 × Return(t)² + 0.94 × Variance(t − 1)
```

For the **0.97** specification:

```text
Variance(t) = 0.03 × Return(t)² + 0.97 × Variance(t − 1)
```

The workbook initializes both variance series with:

```text
Initial Variance = 0.0001
```

This provides a starting value for the recursive calculation. After the initial observation, each day's variance depends on the previous day's estimated variance and the current portfolio return.

The formulas are then applied sequentially across the full historical dataset.

---

## What EWMA Measures

EWMA volatility is designed to capture **changing market volatility over time**.

The key difference compared with a simple standard deviation is the way historical observations are weighted.

Under EWMA:

* Recent returns receive greater influence.
* Older returns gradually lose influence.
* Large recent returns can cause the volatility estimate to increase.
* A period of relatively small returns can cause the estimated volatility to decline over time.

This means EWMA can react to changes in the current market environment without completely discarding historical information.

For example, if the portfolio experiences several unusually large daily returns, the EWMA variance will increase because the squared returns enter the calculation with additional weight.

If subsequent returns become smaller, the estimated variance will gradually decline as the previous high-volatility observations receive progressively less weight.

---

## Decay Factor (λ)

The decay factor controls how quickly the model responds to new information.

The two values used in this workbook are:

```text
λ = 0.94
λ = 0.97
```

A **lower λ** places relatively more weight on the current return and therefore produces a more responsive volatility estimate.

A **higher λ** places relatively more weight on the previous variance estimate and therefore produces a smoother, slower-moving volatility estimate.

Therefore:

* **λ = 0.94** → more responsive to recent market movements
* **λ = 0.97** → more persistent and smoother

This illustrates an important model-design trade-off between **responsiveness and stability**.

A lower decay factor can react more quickly to sudden changes in market conditions, while a higher decay factor can reduce short-term fluctuations in the estimated volatility.

---

## Annualized EWMA Volatility

The workbook converts the daily EWMA variance into annualized volatility using:

```text
Annualized Volatility = √(EWMA Variance × 250)
```

where **250** represents the assumed number of trading days per year.

The resulting columns therefore represent:

* **EWMA Volatility 0.94**
* **EWMA Volatility 0.97**

The volatility estimates are expressed on an annualized basis, making them easier to interpret and compare with conventional annualized volatility measures.

For example, if the EWMA variance increases following a period of larger portfolio returns, the corresponding annualized EWMA volatility will also increase.

---

## Why EWMA Volatility Is Useful

EWMA volatility is commonly useful when risk is **not constant through time**.

Financial markets frequently move between periods of:

* Low volatility
* Normal market conditions
* Elevated volatility
* Short periods of extreme market stress

A fixed volatility estimate may not reflect these changes adequately.

EWMA provides a simple way of creating a **dynamic volatility estimate** that updates every day as new portfolio returns become available.

This makes it useful for areas such as:

* Portfolio risk monitoring
* Dynamic risk limits
* Position sizing
* Margin and collateral analysis
* VaR models
* Stress testing
* Volatility-based exposure management
* Trading and risk dashboards

For example, a risk-management system could use an increase in EWMA volatility as an indication that the portfolio's current market environment has become more volatile, potentially requiring closer monitoring or adjustments to risk limits.

---

## Relationship Between EWMA and VaR

EWMA volatility can also be used as an input into volatility-based VaR models.

The VaR section of this project uses several different approaches, including historical and parametric VaR.

EWMA provides another way of estimating the portfolio's current volatility by giving greater importance to recent observations.

A parametric VaR framework could, for example, replace a constant historical standard deviation with an EWMA volatility estimate:

```text
VaR = Mean Return − Z × EWMA Volatility
```

This would allow the VaR estimate to adjust as the portfolio's estimated volatility changes over time.

The current workbook keeps the EWMA analysis as a separate section so that the methodology can be examined independently from the existing VaR calculations.

---

## Interpretation of the Two EWMA Series

The difference between the two volatility series illustrates how the decay factor affects risk measurement.

When volatility changes rapidly, the **0.94 series** will generally react more quickly because it assigns more weight to the latest return.

The **0.97 series** generally changes more gradually because a larger proportion of the previous variance estimate is retained.

During periods of stable market conditions, the two estimates may move relatively closely together.

Following a sudden increase in portfolio volatility, the more responsive specification can show a sharper adjustment, while the higher-λ specification tends to produce a smoother transition.

This comparison demonstrates that the choice of λ is an important modeling assumption rather than simply a technical parameter.

---

# 3. Stress Testing

The **Stress** sheet evaluates the portfolio under predefined market shock scenarios.

The scenarios include:

### Metals Crash

Applies negative shocks to precious metals, particularly gold and silver, while leaving unrelated instruments unchanged.

### USD Surge

Applies a strengthening-US-dollar scenario across relevant FX and cross-asset exposures.

### Equity Crash

Applies negative shocks to equity indices.

### Global Risk-Off

Represents a broader market stress environment involving simultaneous adverse moves across multiple asset classes.

---

## Custom Stress Scenario

A custom scenario table contains every instrument in the portfolio and allows the user to specify the expected percentage move for each asset.

This allows users to create their own stress scenarios by entering arbitrary positive or negative shocks.

The resulting portfolio P&L is then calculated from the scenario assumptions.

---

# 4. Correlation

The **Correlation** sheet contains a correlation matrix showing the historical relationship between the portfolio instruments.

The matrix calculates the pairwise correlation coefficient for each combination of assets.

Correlation coefficients range from:

```text
+1  = Perfect positive correlation
 0  = No linear correlation
-1  = Perfect negative correlation
```

The matrix is presented as a **conditional-formatting correlation heatmap**, making stronger positive and negative relationships easier to identify visually.

This can help highlight potential diversification effects as well as concentrations where multiple positions may respond similarly to market movements.

---

# 5. Scenario Analysis

The **Scenarios** sheet uses a different portfolio composition for demonstration purposes:

* EURUSD
* Dow Jones
* Gold
* USDJPY
* NASDAQ

Unlike the VaR sheet, this analysis includes a mixture of **long and short exposures**.

The model also incorporates:

* Position exposure
* Contract size
* Notional value
* 14-day ATR
* Direction of exposure
* Scenario volatility assumptions

---

## Volatility Cases

Three volatility environments are included:

### Normal Case

Uses the standard **14-day ATR**.

```text
ATR Multiplier = 100%
```

### Volatile Case

Assumes a more extreme trading environment using:

```text
ATR Multiplier = 150%
```

### Non-Volatile Case

Assumes reduced price movement:

```text
ATR Multiplier = 75%
```

These assumptions allow the same portfolio to be evaluated under different volatility conditions.

---

## Live Scenario Selector

Cell **H2** acts as the scenario selector:

```text
1 = Normal Case
2 = Volatile Case
3 = Non-Volatile Case
```

Changing H2 updates the **Live Case** table.

The Live Case estimates the resulting P&L in USD if each instrument moves up or down according to the selected ATR-based volatility assumption and the direction of the position.

This allows the user to quickly switch between different volatility environments without manually changing each assumption.

---

## Directional Market Scenarios

The workbook also evaluates combinations of USD and equity-index movements:

* **USD Increase + Indices Decrease**
* **USD Decrease + Indices Increase**
* **USD Increase + Indices Increase**
* **USD Increase + Indices Decrease**

These scenarios are useful because the portfolio contains a combination of **USD-related FX positions, indices and gold**, meaning the same directional market movement can produce very different P&L outcomes depending on the exposure mix.

---

## Interactive Random Up/Down Analysis

An additional interactive table allows the user to select whether each instrument moves:

```text
UP
or
DOWN
```

The model then calculates the resulting portfolio P&L.

These scenario tables are also linked to the **Live Case selector**, meaning the assumed movement size changes automatically depending on whether the model is in the Normal, Volatile or Non-Volatile case.

This creates a simple interactive framework for exploring how different combinations of market direction and volatility affect portfolio risk.

---

# Methodologies Used

| Risk Measure / Analysis    | Method                                             |
| -------------------------- | -------------------------------------------------- |
| Historical VaR & CVaR      | Historical percentile of portfolio returns         |
| Parametric VaR & CVaR      | Mean / standard deviation with normal distribution |
| Monte Carlo VaR & CVaR     | Simulated return distribution                      |
| Holding-period sensitivity | Square-root-of-time scaling                        |
| Rolling risk               | 60-day Historical VaR & CVaR                       |
| EWMA volatility            | Exponentially weighted recursive variance          |
| EWMA annualization         | Daily variance × 250 trading days                  |
| Stress testing             | Predefined and custom market shocks                |
| Correlation                | Pairwise historical Pearson correlation            |
| Scenario analysis          | ATR-based directional P&L analysis                 |

---

# Key Assumptions & Limitations

This project intentionally simplifies several aspects of real-world risk modeling.

### Historical Data

The VaR analysis uses approximately **250 historical observations**, which is suitable for demonstration but relatively small for a production risk system.

### Monte Carlo

The Monte Carlo section uses **250 simulated observations** for simplicity and demonstration.

A production implementation would generally use substantially more simulations.

### EWMA Initialization

The EWMA variance calculation begins with an assumed initial variance of:

```text
0.0001
```

The initial value affects the early observations in the recursive series. As more observations are incorporated, the influence of the initial value progressively decreases.

### EWMA Decay Factor

The model uses **λ = 0.94** and **λ = 0.97** as illustrative decay factors.

Different assets, portfolios and risk-management frameworks may use different decay factors depending on the desired responsiveness of the volatility estimate.

The choice of λ therefore represents a model assumption and should be validated when used in a production risk system.

### EWMA Return Assumption

The EWMA calculation uses squared portfolio returns to estimate variance.

It is therefore sensitive to large return observations, since the return is squared before entering the variance calculation.

### Normality Assumption

The Parametric VaR and CVaR calculations assume normally distributed returns.

Financial returns can exhibit skewness, kurtosis and fat tails that are not fully captured by this assumption.

### Holding-Period Scaling

The square-root-of-time approach assumes that returns are sufficiently independent and identically distributed.

This may not hold during periods of market stress.

### Position Direction

The initial VaR example assumes all exposures are long.

Short exposures require the relevant asset return to be reversed before calculating the portfolio return.

### Scenario Analysis

The stress and scenario assumptions are hypothetical and are intended to demonstrate the mechanics of portfolio risk analysis rather than predict actual future market movements.

### Production Risk Management

The workbook is intended as an educational and portfolio demonstration.

A production risk-management framework would typically incorporate additional considerations such as larger datasets, real-time market data, liquidity risk, transaction costs, model validation, backtesting, limit monitoring, data-quality controls and more comprehensive stress-testing methodologies.
