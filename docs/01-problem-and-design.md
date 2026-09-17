# Problem, Data, and System Design

## Problem identification and ideation

This project began initally as an infrastructure exercise. The original brief was to build an
end-to-end machine learning system, data pipeline, training pipeline, inference
service, interactive frontend, model accuracy was listed as secondary.
That framing did not survive first contact with the problem. Forecasting is a case where
the modelling question is hard and genuinely measurable, and it became
clear early that a pipeline moving a bad model around on a fixed schedule would
demonstrate very little. So the question was reframed, Can a neural network learn the
atmosphere's six-hour state-to-state map directly from past observations? Well
enough to beat the trivial forecasts a meteorologist would demand it beat? Framing
it that way forced a definition of success before any model existed. I wanted
forecasts within roughly 1–4 °C of observed, and I set a formal acceptance gate,
the first model had to beat both persistence ("tomorrow looks like today") and
climatology ("the historical average for this date and hour") on
latitude-weighted temperature RMSE at 24, 48, and 72 hours. Beating climatology at
168 hours was named a stretch goal rather than a requirement, because published
state-of-the-art models barely manage it for surface temperature. Writing that gate
down in advance is the single decision that most shaped the project, because it
meant every later result could be judged against a standard I had not tuned to fit.

## Data selection

ERA5 reanalysis was the obvious foundation. Its a consistent, hourly, gridded
reconstruction of the atmosphere built by combining historical observations with a
physics model, and the surface most published weather-ML work is benchmarked on.
The initial plan used six variables. It settled on ten, chosen so the inputs
actually carry the physics that surface temperature depends on rather than just
correlating with it upper-air temperature, wind, and geopotential height for
advection and synoptic state, sea-level pressure, dewpoint for humidity, cloud
cover and downward solar radiation for the radiative budget, and soil moisture,
which governs how incoming energy splits between heating the air and evaporating
water. Thirty years (1995–2024) were taken at 0.25° over the continental US and a
margin of surrounding ocean and terrain, and split by year training,
validation, and test, rather than randomly, so no forecast is ever evaluated
against a period the model trained on. One data quality finding is worth recording
because it shaped how I validate data generally, soil moisture came back with
near-zero values over the ocean instead of missing values, which silently dragged
its accumulated mean well below any physically plausible figure. Shape checks and
NaN counts would never have caught it. Comparing accumulated statistics against
physical expectation did, and the fix was to derive an explicit land–sea mask and
exclude ocean cells from that variable's statistics.

## Solution design

The original system design was a live product: fetch the newest observations every
24 hours, run inference, serve forecasts through an API, retrain every 14 days, with an automatic promotion gate, all on managed cloud services at roughly
fifty dollars a month. It was abandoned, and for a reason that no amount of
engineering could fix, ERA5 is published about five days behind real time.
A model trained on it cannot be given today's atmosphere, so a live "current
weather" product was never actually available on this data. Rather than paper over
that, I inverted it. Every forecast is initialized from a historical date, which
means every forecast can be shown next to the observations that verified it,
the prediction, the truth, and the error, with the error visibly growing as lead
time increases. That is a more honest artifact than a live forecast would have
been, and a more interesting one, because it puts the model's failures on screen
instead of hiding them. The pivot also collapsed the infrastructure. A
pre-computed catalog of scored forecasts is a set of static files; a seven-day
forecast compresses to under a megabyte, and the work that produces it is a batch
job, not a service. The API server, container registry, orchestration, and
monitoring stack all became unnecessary, and running cost fell from about fifty
dollars a month to the price of a domain name. The general lesson is one I would
apply again, the properties of the data dictated the correct system architecture,
and recognizing that early deleted most of the system.
