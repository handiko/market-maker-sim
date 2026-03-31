# Avellaneda-Stoikov Market Maker Simulator

An interactive, high-fidelity implementation of the seminal **Avellaneda-Stoikov (2008)** model for algorithmic market making. This simulator explores the stochastic control problem of managing inventory risk while providing liquidity in a limit order book.

## Overview

The simulator models a market maker (MM) who provides liquidity by simultaneously posting bid and ask quotes. The MM's goal is to capture the spread while managing **Inventory Risk** (the risk of price movement against a held position) and **Adverse Selection** (the risk of trading against informed participants).

The system runs a real-time loop (160ms ticks) simulating a continuous market environment using Geometric Brownian Motion for price action and Poisson processes for trade arrivals.

---

## Theoretical Foundation

The simulator is built on three core mathematical pillars:

### 1. The Reservation Price ($r$)
Unlike a naive trader who quotes around the mid-price $s$, the Avellaneda-Stoikov MM calculates a "fair value" adjusted for their current inventory $q$. 

$$r(s, q, t) = s - q \gamma \sigma^2 (T - t)$$

* **$s$**: Current Mid-Price.
* **$q$**: Inventory position (positive for long, negative for short).
* **$\gamma$**: Risk aversion parameter.
* **$\sigma$**: Market volatility.
* **$(T-t)$**: Remaining time horizon.

**Logic:** If the MM is "Long" ($q > 0$), the reservation price drops below the mid-price. This shifts both bid and ask quotes downward, making it more likely to sell (liquidate) and less likely to buy more.

### 2. The Optimal Spread ($\delta$)
The model calculates the optimal distance from the reservation price to place the bid and ask quotes:

$$\delta = \gamma \sigma^2 (T - t) + \frac{2}{\gamma} \ln\left(1 + \frac{\gamma}{k}\right)$$

The total spread is comprised of an **Inventory Risk Premium** and an **Adverse Selection Premium**. As volatility ($\sigma$) or risk aversion ($\gamma$) increases, the spread widens to compensate the MM for the higher risk.

### 3. Trade Arrival (Poisson Process)
The probability of a quote being "hit" (a fill) is modeled as a decaying exponential function of the distance from the mid-price:

$$P(\text{fill}) = A \cdot \exp(-k \cdot \text{dist}) \cdot \Delta t$$

* **$A$**: Intensity of overall market trades (frequency).
* **$k$**: Market "depth" or price sensitivity. High $k$ means the market is very sensitive; if your price is too far from the mid, you will never get filled.

---

## Software Architecture

The application is a single-file web tool (`index.html`) using **Chart.js** for visualization and vanilla **JavaScript** for the simulation engine.

### Key Components:
- **Price Engine (`randn` function):** Uses the **Box-Muller Transform** to generate Gaussian noise for the Geometric Brownian Motion (GBM) update.
- **Quote Engine (`asQuotes` function):** The mathematical "brain" that calculates $r$ and $\delta$ based on current state and slider inputs.
- **Fill Engine (`poissonProb` function):** Determines fills by comparing a random float against the calculated Poisson intensity for both the bid and ask sides.
- **State Management:** Tracks PnL (Mark-to-Market), Cash, Inventory, and Fills in a centralized loop.

---

## Parameter Guide

| Parameter | Symbol | Impact on Behavior |
| :--- | :--- | :--- |
| **Risk Aversion** | $\gamma$ | Higher values lead to wider spreads and more aggressive "skewing" to keep inventory at zero. |
| **Volatility** | $\sigma$ | Increases the "drift" of the price and widens spreads to account for larger potential price swings. |
| **Order Depth** | $k$ | Represents how "thick" the order book is. Higher $k$ means you must quote very close to the mid-price to get filled. |
| **Order Frequency**| $A$ | Controls the total volume of trades in the market. |
| **Manual Skew** | — | A heuristic override to force the MM to lean long or short regardless of the model's math. |

---

## Analysis & Insights

Based on the research provided in the accompanying essay:

1.  **The Inventory Leash:** Pure spread-capture is impossible without an inventory limit. The reservation price acts as a "leash" that pulls the MM back to a neutral position.
2.  **Regime Awareness:** In high-volatility regimes, the MM must increase $\gamma$ to survive. In low-volatility, "mean-reverting" regimes, the MM can lower $\gamma$ to capture more volume.
3.  **Adverse Selection:** When $k$ is high, the MM is effectively competing in a "winner-take-all" price environment, requiring tighter spreads and faster reaction times.

---

## How to Run

1.  Download the `index.html` file or open this link: [here](https://handiko.github.io/market-maker-sim/)
2.  Open it in any modern web browser (Chrome, Firefox, or Edge recommended).
3.  Adjust the sliders to set your market conditions.

---

## References
* *Avellaneda, M., & Stoikov, S. (2008). High-frequency trading in a limit order book.*
* *Implementation and documentation by Handiko Gesang.*
