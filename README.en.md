🌐 [Español](README.md) · **English**

# Tourism Demand Intelligence (TDI)

**Tourism demand forecasting and simulation for data-driven destination management: how many visitors will come, when capacity saturates, and what a crisis costs.**

> 🔒 This repository is a portfolio showcase. The source code and full methodology are the author's intellectual property and are not published here. A private demo is available on request.

*Figures are in Spanish.*

---

## What does it solve?

TDI answers the questions a destination manager faces every day:

- **How many visitors will come in the next 12 months?** A monthly forecast with a confidence interval.
- **When does capacity saturate?** Demand against the current visitor cap, month by month.
- **What does a crisis cost?** Visitors lost to disasters, a pandemic or protests, and how long recovery takes.
- **What if…?** The visitor cap changes, demand grows or falls, a crisis repeats, or access management gets worse.

## The ecosystem

```mermaid
flowchart LR
    F["Destination sources<br/>(official reports, climate,<br/>primary data)"] --> A["Adapters<br/>(one per project)"]
    A --> E["TDI standard<br/>TDI_Destino · TDI_Serie<br/>Variable catalog"]
    E --> M["TDI engine<br/>EDA · Machine Learning · Scenarios"]
    M --> P["Power BI dashboard<br/>(in development)"]
    M --> S["Web simulator<br/>(in development)"]
```

Each destination delivers its data in a **common standard**. From there the engine is the same for all of them: it contains no destination names, and everything specific lives in the project configuration.

## One engine, any destination

The engine diagnoses which data modules each destination has and **adapts**: it runs whatever the data supports and explains what it skips. Two demonstration cases with opposite profiles:

| Module | Machu Picchu (Peru) | Salar de Uyuni (Bolivia) |
|---|---|---|
| CORE (monthly visitors) | Complete | Partial |
| M1 Demand (domestic / international) | Partial | No data |
| M2 Access | No data | No data |
| M3 Climate | Complete | Complete |
| M4 Economy (spending) | No data | Partial |
| M5 Disruptive events | Complete | Partial |
| M6 Capacity (visitor cap) | Partial | No data |

Machu Picchu has **capacity** data but no economic data; the Salar is the other way around. The same code produces, for each one, the analyses its data can support.

**The engine does not invent data.** The Salar series has data for 62 of 108 months (2016–2024). The gaps stay as gaps; they are not filled with zeros or estimates:

![Salar de Uyuni monthly series with gaps and pandemic](assets/salar_01_serie_eventos.png)

The automated tests also run a **synthetic destination** through the whole pipeline. It has only the three minimum variables (destination, period, visitors), and the engine completes EDA, ML and scenarios while explaining every analysis it cannot run.

## Machu Picchu case: the destination in data

Monthly visitors to the Inca City (2002-01 to 2026-08, 296 months), with its disruptive events coded:

![Machu Picchu monthly series with events](assets/mp_01_serie_eventos.png)

| Seasonality | STL decomposition |
|---|---|
| ![Seasonality](assets/mp_03_estacionalidad.png) | ![STL decomposition](assets/mp_04_descomposicion_stl.png) |

Capacity use against the regulatory daily visitor cap:

![Capacity use](assets/mp_07_capacidad.png)

## Honest Machine Learning

Four models are compared against a demanding baseline: **Seasonal Naïve**, which repeats the same month of the previous year. Validation is **temporal and ex-ante**: each test year is predicted using only the past, and events and climate in the forecast period are treated as unknown.

**Backtesting** (6 test years: 2017, 2018, 2019, 2023, 2024 and 2025):

| Model | MAPE in normal months | Mean MAPE |
|---|---|---|
| **Random Forest** (selected) | **12.58%** | 23.65% |
| Seasonal Naïve (baseline) | 13.90% | 18.57% |
| Gradient Boosting | 14.02% | 24.30% |
| Multiple linear regression | 14.55% | 19.63% |

Random Forest cuts the baseline's error in normal months by 9%.

![Error by test year](assets/mp_08_backtesting.png)

**Blind test, January–August 2026.** These are 8 months that no model saw during training:

| Model | MAPE | MAE | R² |
|---|---|---|---|
| Seasonal Naïve | **8.3%** | 8,200 | 0.864 |
| Random Forest | 16.4% | 15,634 | 0.509 |
| Gradient Boosting | 17.5% | 17,111 | 0.440 |
| Multiple linear regression | 19.9% | 20,965 | 0.309 |

Random Forest forecast 1,041,673 visitors and 916,599 were recorded (+13.6%). **All four models overestimated all eight months**, the baseline included.

![2026 blind test](assets/mp_09_prueba_final.png)

**Interpretation.** The 2026 series came in below what the trend anticipated. The gap is consistent with supply constraints documented that year in the ticketing system. According to press reports, an audit report found that 62.9% of 209,051 reservations analyzed ended up cancelled [1], the Government attributed the drop in visitors to problems with the ticketing platform [2], and the end of advance booking was announced in September 2026 [1]. **The model estimates demand, and the difference points to factors holding the destination back.**

A 12-month forecast with a 90% interval, and a counterfactual showing how many visitors there would have been without the disruptive events:

| Forecast | Counterfactual |
|---|---|
| ![Forecast](assets/mp_10_pronostico.png) | ![Counterfactual](assets/mp_11_contrafactual.png) |

## Events and scenarios

**Event impact.** Visitors lost compared with what the previous normal years predicted:

| Event | Period | Months | Loss |
|---|---|---|---|
| Natural disaster | 2010-02 to 2010-03 | 2 | 100% |
| Pandemic | 2020-03 to 2021-12 | 22 | 81.4% |
| Social protests | 2022-12 to 2023-02 | 3 | 70.8% |

![Event impact](assets/mp_05_impacto_eventos.png)

**A shock library calibrated on the destination's own history.** Each event contributes its magnitude, its duration and its month-by-month recovery curve. For example, after the protests the destination took 9 months to return to 90% of expected levels. The library feeds the scenarios: "what if the protests repeat in 2028?" is simulated with the real footprint they left, not with a generic assumption.

**Scenarios to 2030.** Growth is derived from the destination's historical rate and combined with capacity levers, shocks and management friction:

| Scenario (2030) | Demand | Admitted visits | Unmet demand |
|---|---|---|---|
| Conservative | 1,499,865 | 1,499,865 | 0 |
| Base | 1,668,501 | 1,664,001 | 4,500 |
| Optimistic | 1,850,418 | 1,739,188 | 111,230 |
| Expanded cap | 1,668,501 | 1,668,501 | 0 |
| Management friction (7%) | 1,668,501 | 1,551,706 | 116,795 |

- **Protests 2028:** repeating the 2022–23 shock cuts 2028 demand by 33.6% compared with Base.
- **Capacity:** with high growth, the visitor cap becomes the binding constraint around 2028–2030 (≈111 thousand unmet visits in 2030, 6%). In the base scenario it barely starts to bind in 2030 (0.3%). Expanding the cap removes that gap.

| Annual scenarios | Demand vs. capacity |
|---|---|
| ![Annual scenarios](assets/mp_13_escenarios_anuales.png) | ![Capacity by scenario](assets/mp_14_capacidad_escenarios.png) |

## Quality and rigor

- **28 automated tests:**
  - data contract;
  - genericity (the engine mentions no destination);
  - synthetic destination;
  - scenario simulator;
  - model regression.
- **Ex-ante temporal validation:** no model knows the future it predicts.
- **Traceability:** every coded event and every capacity period records its source and whether it has been verified.
- **Reproducibility:** the standard data is regenerated from the original sources with a single command, with no manual steps.

## Products

| Product | Status |
|---|---|
| Analytics engine (EDA, ML, scenarios) | ✔ v0.4 |
| Power BI dashboard | In development |
| Web scenario simulator | In development |

## Stack

Python (pandas, scikit-learn, statsmodels) · Power BI · Streamlit

## Roadmap

- ✔ **v0.4:** generic engine, adaptation layer and automated tests
- ◻ Power BI dashboard
- ◻ Web simulator

## Data sources

- **Machu Picchu:** Peru's Ministry of Culture, Decentralized Directorate of Culture of Cusco, compiled by MINCETUR (visitor arrivals); official notes and press (events); capacity resolutions and protocols.
- **Climate:** NASA POWER (MERRA-2).
- **Salar de Uyuni:** the author's primary data.

> A demonstration case built with public data. It is not an official forecast and does not represent the position of any institution.

## References

1. Latin Times, October 2026: "Informe de Auditoría N° 007-2026" (62.9% of 209,051 reservations cancelled) and the end of advance booking announced on September 11, 2026. https://latintimes.com/more-90000-machu-picchu-tickets-open-giving-travelers-new-chance-visit-before-2026-year-end-599967
2. América TV, September 2026: the Government attributes the drop in visitors to problems with the ticketing platform. https://www.americatv.com.pe/noticias/actualidad/machu-picchu-gobierno-anuncia-nuevo-sistema-venta-entradas-ante-caida-visitantes-n522565

## Author

**Víctor Hugo Villegas Ríos**, freelance data analytics and data science consultant.
