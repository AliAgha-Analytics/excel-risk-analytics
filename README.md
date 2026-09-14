# Exposure Risk Analytics

An Excel-based trading portfolio risk analysis project covering **Value at Risk (VaR), Conditional VaR (CVaR), stress testing, correlation analysis, volatility-based sensitivity analysis and scenario analysis**.

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

The workbook contains four main sheets:

1. **VaR** – Historical, Parametric and Monte Carlo VaR/CVaR, holding-period sensitivity and rolling risk
2. **Stress** – Multi-asset stress-testing scenarios
3. **Correlation** – Correlation analysis across instruments
4. **Scenarios** – ATR-based scenario analysis with long/short exposures and interactive case selection

---

# 1. VaR

The **VaR** sheet contains approximately **250 trading days of historical returns** for EURUSD, XAUUSD, XAGUSD, USDJPY, S&P 500 and NASDAQ.

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

# 2. Stress Testing

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

# 3. Correlation

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

# 4. Scenario Analysis

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
| Stress testing             | Predefined and custom market shocks                |
| Correlation                | Pairwise historical Pearson correlation            |
| Scenario analysis          | ATR-based directional P&L analysis                 |

---

# Key Assumptions & Limitations

This project intentionally simplifies several aspects of real-world risk modeling.

### Historical Data

The VaR analysis uses approximately **250 historical observations**, which is suitable for demonstration but relatively small for a production risk system.

### Monte Carlo

The Monte Carlo section uses **250 simulated observations** for simplicity and demonstration. A production implementation would generally use substantially more simulations.

### Normality Assumption

The Parametric VaR and CVaR calculations assume normally distributed returns. Financial returns can exhibit skewness, kurtosis and fat tails that are not fully captured by this assumption.

### Holding-Period Scaling

The square-root-of-time approach assumes that returns are sufficiently independent and identically distributed. This may not hold during periods of market stress.

### Position Direction

The initial VaR example assumes all exposures are long. Short exposures require the relevant asset return to be reversed before calculating the portfolio return.

### Scenario Analysis

The stress and scenario assumptions are hypothetical and are intended to demonstrate the mechanics of portfolio risk analysis rather than predict actual future market movements.

---
