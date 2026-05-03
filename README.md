# cox-delivery-time-survival-analysis
Survival analysis project using the Cox proportional hazards model to estimate delivery time behavior under different logistics scenarios.

Post: https://www.linkedin.com/posts/brenogdsilva_datascience-survivalanalysis-machinelearning-activity-7454544907383320576-Q9pP?utm_source=share&utm_medium=member_desktop&rcm=ACoAADzv2l0B8U0wmrrJ29QSB7uUMS1y_sAZlBo

# Cox Delivery Time Survival Analysis

This repository presents an educational application of survival analysis to logistics data. The goal is to model the time until a delivery occurs and understand how operational factors affect the expected delivery dynamics.

Instead of treating delivery performance only as a binary outcome, such as delivered or not delivered, this project focuses on time-to-event modeling. In this context, the event of interest is the delivery of an order, and the response variable is the time until that delivery happens.

The project uses simulated logistics data, fits a Cox proportional hazards model, estimates hazard ratios, and creates an animated visualization comparing predicted survival curves across different operational scenarios.

---

## Introduction

Logistics operations are naturally time-dependent. Delivery performance is not only about whether an order is delivered, but also about how long it takes until delivery occurs.

This type of problem can be studied using survival analysis, a statistical framework designed to model time until an event. Although survival analysis originated in biomedical and reliability studies, it is also useful in business and operational contexts, such as delivery time modeling, churn timing, failure analysis, customer retention, and service-level monitoring.

The Cox proportional hazards model, introduced by Cox (1972), is one of the most important methods in survival analysis. It allows the relationship between explanatory variables and the event rate to be modeled without requiring a fully specified baseline hazard function.

In this project, the Cox model is used to estimate how logistics factors such as distance, express shipping, holidays, risky carriers, and weekend dispatch affect the time until delivery.

---

## Problem Context

In logistics analytics, a company may want to answer questions such as:

* Which factors make a delivery more likely to happen sooner?
* Which factors are associated with slower deliveries?
* How does express shipping change the expected delivery curve?
* How does operational risk accumulate when holidays, risky carriers, and weekend dispatch occur together?
* Can survival analysis provide a better view of delivery behavior than a simple average delivery time?

Traditional analysis often compares average delivery times across groups. However, this can hide important information about the timing of events and censoring.

Survival analysis is useful because it works directly with the time until the event and can handle observations where the event has not yet occurred during the observation window.

---

## Objective

The objective of this project is to simulate a logistics dataset and estimate the effect of operational variables on delivery time using a Cox proportional hazards model.

The event of interest is:

```text
Delivery completed
```

The time variable is:

```text
Number of days until delivery
```

The event indicator is:

```text
1 if the delivery occurred
0 if the delivery was censored
```

Censoring means that the delivery time was not fully observed before the observation period ended.

---

## Methodological Background

Let $T$ be a non-negative random variable representing the time until delivery.

The survival function is defined as:

$$
S(t) = P(T > t)
$$

In this project, $S(t)$ represents the probability that an order has not yet been delivered after time $t$.

The hazard function is defined as:

$$
h(t) = \lim_{\Delta t \to 0} \frac{P(t \le T < t + \Delta t \mid T \ge t)}{\Delta t}
$$

The hazard function represents the instantaneous event rate at time $t$, conditional on the order not having been delivered before time $t$.

The Cox proportional hazards model is written as:

$$
h(t \mid x) = h_0(t)\exp(\beta_1x_1 + \beta_2x_2 + \cdots + \beta_px_p)
$$

where:

* $h(t \mid x)$ is the hazard function for an order with covariates $x$;
* $h_0(t)$ is the baseline hazard function;
* $x_1, x_2, \ldots, x_p$ are explanatory variables;
* $\beta_1, \beta_2, \ldots, \beta_p$ are model coefficients.

The hazard ratio for a one-unit increase in variable $x_j$ is:

$$
HR_j = \exp(\beta_j)
$$

The interpretation is:

* $HR_j > 1$ means the delivery event tends to happen sooner;
* $HR_j < 1$ means the delivery event tends to happen later;
* $HR_j = 1$ means no estimated effect on the delivery rate.

---

## Data Generating Process

The dataset is synthetically generated to represent logistics operations.

The simulated variables are:

* `dist_km`: delivery distance in kilometers;
* `expressa`: whether the shipment is express;
* `feriado`: whether dispatch happened during a holiday;
* `carrier_risco`: whether the carrier is considered operationally risky;
* `despacho_fds`: whether dispatch happened during the weekend.

Distance is converted into units of 100 kilometers:

$$ dist\_100km = \frac{dist\_km}{100} $$

The synthetic linear predictor is defined as:

$$ \eta_i = \beta_1 dist\_100km_i + \beta_2 expressa_i + \beta_3 feriado_i + \beta_4 carrier\_risco_i + \beta_5 despacho\_fds_i $$

The coefficients used in the simulation are:

```text
dist_100km     = -0.18
expressa       =  0.55
feriado        = -0.35
carrier_risco  = -0.45
despacho_fds   = -0.20
```

The event time is generated from an exponential time-to-event mechanism:

$$
T_i = \frac{-\log(U_i)}{\lambda_0 \exp(\eta_i)}
$$

where:

* $U_i$ is sampled from a uniform distribution between 0 and 1;
* $\lambda_0$ is the baseline event rate;
* $\eta_i$ is the linear predictor.

The censoring time is generated independently:

$$
C_i \sim Uniform(5.5, 10.0)
$$

The observed time is:

$$
Y_i = \min(T_i, C_i)
$$

The event indicator is:

$$
\delta_i = I(T_i \le C_i)
$$

where $\delta_i = 1$ means the delivery was observed and $\delta_i = 0$ means the observation was censored.

---

## Methodology

The project follows the steps below:

* Simulate 700 logistics observations.
* Generate delivery times based on a time-to-event process.
* Generate censoring times.
* Build the final modeling dataset.
* Fit a Cox proportional hazards model using `lifelines`.
* Estimate hazard ratios and confidence intervals.
* Create predicted survival curves for selected logistics scenarios.
* Build an animation showing model interpretation and scenario comparison.

The model is fitted using the following variables:

```text
tempo
evento
dist_100km
expressa
feriado
carrier_risco
despacho_fds
```

The response structure is composed of:

* `tempo`: observed time;
* `evento`: event indicator.

The explanatory variables are:

* `dist_100km`;
* `expressa`;
* `feriado`;
* `carrier_risco`;
* `despacho_fds`.

---

## Model

The Cox model is estimated with the `CoxPHFitter` class from the `lifelines` package.

```python
cph = CoxPHFitter()
cph.fit(model_df, duration_col="tempo", event_col="evento")
```

After fitting the model, the coefficients are transformed into hazard ratios:

$$
HR_j = \exp(\hat{\beta}_j)
$$

Confidence intervals are also transformed to the hazard ratio scale:

$$
HR_{low,j} = \exp(\hat{\beta}_{low,j})
$$

$$
HR_{high,j} = \exp(\hat{\beta}_{high,j})
$$

This makes the results easier to interpret in operational terms.

---

## Scenarios

The project compares three predicted delivery scenarios.

### Standard Scenario

```text
Distance: 350 km
Express shipping: No
Holiday dispatch: No
Risky carrier: No
Weekend dispatch: No
```

### Critical Scenario

```text
Distance: 550 km
Express shipping: No
Holiday dispatch: Yes
Risky carrier: Yes
Weekend dispatch: Yes
```

### Express Scenario

```text
Distance: 350 km
Express shipping: Yes
Holiday dispatch: No
Risky carrier: No
Weekend dispatch: No
```

For each scenario, the model predicts a survival curve:

$$
S(t \mid x) = P(T > t \mid x)
$$

In this project, this curve represents the probability that the order has not yet been delivered by time $t$.

---

## Results

The fitted model produces hazard ratios for the logistics variables.

The expected interpretation follows the simulation design:

* Express shipping tends to increase the hazard of delivery.
* Longer distance tends to reduce the hazard of delivery.
* Holiday dispatch tends to reduce the hazard of delivery.
* Risky carriers tend to reduce the hazard of delivery.
* Weekend dispatch tends to reduce the hazard of delivery.

In practical terms, a higher hazard means the delivery event is more likely to occur sooner. A lower hazard means the order tends to remain undelivered for longer.

The animated visualization contains two panels.

The first panel displays the estimated hazard ratios and their confidence intervals. Variables with hazard ratios above 1 are associated with faster delivery. Variables with hazard ratios below 1 are associated with slower delivery.

The second panel displays predicted survival curves for the three scenarios. The express scenario tends to show a faster decrease in the survival curve, meaning the probability of remaining undelivered decreases more quickly. The critical scenario tends to keep a higher survival probability over time, meaning the order remains more likely to be undelivered as time passes.

---

## Interpretation

The project shows how survival analysis can be used to understand delivery timing rather than only delivery status.

A binary model could estimate whether an order was delivered or not. However, the Cox model allows the analyst to study when the event happens and how each operational factor changes the event rate.

For logistics teams, this type of approach can support:

* service-level monitoring;
* delivery risk analysis;
* carrier performance evaluation;
* operational prioritization;
* express shipping impact analysis;
* holiday and weekend planning;
* scenario simulation.

The main advantage is that the model produces interpretable measures through hazard ratios.

---

## Limitations

This project is educational and uses synthetic data.

The main limitations are:

* the dataset is simulated;
* the event time follows a simplified exponential mechanism;
* censoring is generated independently;
* the proportional hazards assumption is not formally tested;
* no real carrier or route information is used;
* no geographical clustering is considered;
* only a small set of logistics variables is included.

In a real-world application, the analysis could be improved by adding:

* real delivery tracking data;
* carrier-level historical performance;
* route-level information;
* customer region;
* warehouse location;
* promised delivery date;
* weather variables;
* traffic variables;
* operational capacity indicators;
* tests of proportional hazards;
* model validation with out-of-time samples.

---

## Conclusion

This project demonstrates how survival analysis can be applied to logistics delivery-time modeling.

The Cox proportional hazards model provides an interpretable framework for estimating how operational variables affect the time until delivery. Instead of analyzing only average delivery time or final delivery status, the model focuses on the timing of the delivery event and the probability of remaining undelivered over time.

The results show that express shipping increases the tendency of earlier delivery, while longer distance, holiday dispatch, risky carriers, and weekend dispatch are associated with slower delivery dynamics in the simulated data.

The animation helps communicate these effects visually by combining hazard ratios and scenario-based survival curves.

---

## References

Cox, D. R. (1972). Regression Models and Life-Tables. Journal of the Royal Statistical Society: Series B, 34(2), 187-220. https://doi.org/10.1111/j.2517-6161.1972.tb00899.x

Kaplan, E. L., & Meier, P. (1958). Nonparametric Estimation from Incomplete Observations. Journal of the American Statistical Association, 53(282), 457-481. https://doi.org/10.1080/01621459.1958.10501452

Andersen, P. K., & Gill, R. D. (1982). Cox's Regression Model for Counting Processes: A Large Sample Study. The Annals of Statistics, 10(4), 1100-1120. https://doi.org/10.1214/aos/1176345976

Davidson-Pilon, C. (2019). lifelines: survival analysis in Python. Journal of Open Source Software, 4(40), 1317. https://doi.org/10.21105/joss.01317
