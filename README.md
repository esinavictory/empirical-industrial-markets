# empirical-industrial-markets
# Demand Estimation in a Stylized Pay-TV Market with PyBLP

Project assignment in Empirical Industrial Organization, Fall 2026, HSE University (St. Petersburg).

## Overview

This project estimates demand for pay-TV services in a simulated market and compares three discrete choice models of increasing flexibility: **plain logit**, **nested logit**, and **mixed (random coefficients) logit**. All models are estimated with the [PyBLP](https://pyblp.readthedocs.io) package.

The exercise covers the main steps of a standard demand estimation workflow in IO: handling price endogeneity with instruments, computing own- and cross-price elasticities, simulating market entry, and recovering the distribution of consumer valuations.

## Data

The dataset contains 1,000 geographical markets (`market_ids` from 0 to 999), each with four inside goods and an outside option. Each good is produced by a separate firm.

| Variable | Description |
|---|---|
| `market_ids`, `product_ids`, `firm_ids` | Market, product, and firm identifiers |
| `shares` | Market share of the good |
| `prices` | Price of the good |
| `quality` | Observable non-negative quality (related to the number of channels) |
| `satellite`, `wired` | Dummies for service type. Goods 1 and 2 are satellite, goods 3 and 4 are wired |
| `demand_instruments0` | Cost-shifter, a valid instrument for price |

The data file (`data_HA_2026.xlsx`) is provided by the course professor.

## Models

Utility of consumer *i* from good *j* in market *t*:

u<sub>ijt</sub> = β<sup>(1)</sup>·quality + β<sup>(2)</sup>·satellite + β<sup>(3)</sup>·wired − α·price + ξ<sub>jt</sub> + ε<sub>ijt</sub>

There is no constant, so the satellite and wired dummies act as type-specific baseline utilities.

| Model | Specification | Instruments |
|---|---|---|
| Plain logit (IV and OLS) | Gumbel shocks, no heterogeneity | Cost-shifter (IV); price as its own instrument (OLS) |
| Nested logit | Nests by service type (satellite / wired), outside good in its own nest | Cost-shifter + quality of the other good in the same nest |
| Mixed logit | Random coefficients on `satellite` and `wired`, zero covariance | Above + sum of qualities of other goods in the market |

Mixed logit settings: 100 Monte Carlo draws per market, seed 0, BFGS with a 10⁻⁶ gradient tolerance.

## Main results

**Market 0 descriptives.** The outside option has a share of 0.090. Good 1 holds 66.4% of satellite sales. The average price is 2.72 for satellite and 2.37 for wired services.

**Plain logit: IV vs. OLS**

| Parameter | IV | OLS |
|---|---|---|
| Price (α) | 1.988 | 1.095 |
| Quality | 0.943 | 0.851 |
| Satellite | 4.066 | 1.676 |
| Wired | 3.030 | 0.753 |

OLS understates the price sensitivity: prices are positively correlated with unobserved quality shocks (ξ), which biases the price coefficient toward zero. With the IV estimates, satellite service gives higher baseline utility than wired (4.07 vs. 3.03).

**Elasticities (market 0).** The elasticity of good 2's share with respect to good 4's price is 0.755 in plain logit and 0.720 in nested logit. A positive value means the goods are substitutes. Plain logit imposes proportional substitution patterns, while nested logit allows closer substitution within a nest.

**Nested logit.** The nesting parameter is ρ = 0.08 (s.e. 0.06), not significant at the 5% level. There is no strong nesting-by-service-type structure in the data.

**Individual choice and entry prediction**

- A consumer with the shocks given in the assignment chooses good 2.
- A new satellite entrant with average quality and price in market 0 would capture a short-run market share of about **7.9%**.

**Mixed logit**

| Coefficient | Mean | Std. deviation |
|---|---|---|
| Satellite | 4.196 | 1.259 |
| Wired | 3.236 | 0.125 |

Tastes for satellite TV are far more dispersed than for wired TV. A plausible reason is that satellite service depends on housing (a dish needs a clear view of the sky, which suits houses and rural areas better than apartment buildings), while wired service is a more standardized product.

**Willingness to pay for baseline-quality satellite service.** The population average is 2.01, and 95% of consumers have a WTP between 0.83 and 3.20.
