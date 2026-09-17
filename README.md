# ConvTherm — Design Documentation

ConvTherm is a convolutional LSTM that forecasts 2 m air temperature across the
continental United States, seven days ahead, and displays each forecast beside
the observations that actually verified it.

### --> **Live site:** [convtherm.com](https://convtherm.com) <--


This repository is a design discussion, not a code repo. It details how the
project was created, what problem it set out to solve, which decisions were
made and why, which of them turned out to be wrong, and what the evidence was at
each step.

## Contents

| Document | Covers |
|---|---|
| [Problem, data, and system design](https://github.com/mattmcc126/ConvTherm/blob/main/docs/01-problem-and-design.md) | Problem identification and ideation, data selection, solution design |
| [Baseline](https://github.com/mattmcc126/ConvTherm/blob/main/docs/02-baseline.md) | Baseline design, training and testing, and what the results implied |
| [ConvLSTM](https://github.com/mattmcc126/ConvTherm/blob/main/docs/03-convlstm.md) | Architecture, training and testing, rollout fine-tuning |
| [Frontend](https://github.com/mattmcc126/ConvTherm/blob/main/docs/04-frontend.md) | Interface design and delivery |

## Results in brief

Measured on held-out years (2022–2024), latitude-weighted RMSE on 2 m temperature:

| Lead | RMSE | vs. climatology |
|---|---|---|
| 6 h | 0.91 °C | +0.74 |
| 24 h | 1.37 °C | +0.61 |
| 72 h | 3.03 °C | +0.15 |
| 120 h | 3.70 °C | −0.06 |
| 168 h | 4.07 °C | −0.14 |

Skill score is `1 − RMSE_model / RMSE_reference`; positive means the model beats
that reference. The crossover to negative at long leads is expected and is
discussed in document 3, it's a structural property due to a limited-area model,
not a real training deficiency.

## A note on scope

This is a personal research project, not an operational forecasting system. The
forecasts on the live site are initialized from historical dates and describe
weather that has already happened. Nothing here should be used to make a decision
that depends on knowing the weather.
