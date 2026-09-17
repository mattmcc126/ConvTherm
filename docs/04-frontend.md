# Frontend

## Design

The interface's job was set by a property of the data rather than taste. Because
ERA5 lags real time by about five days, every forecast is initialized from a
historical date, meaning every forecast can be placed beside the observations
that verified it. The interface is built around that idea with three switchable
layers (the forecast, what actually happened, and the error between them), a
scrubber that steps through the seven-day rollout six hours at a time, and a chart
of error against lead time showing precisely where the model stops beating a
climatological average. Presenting the model's failures as a feature,
rather than showing only the forecast, is the entire point of the product. Colour
carries most of the information, so it was treated as an engineering problem rather
than a styling one. The temperature layers and the error layer use two different
scales because they presenet different tempearture ranges. A multi-hue meteorological
ramp spanning −60 °F to 120 °F for absolute temperature, where fine gradient
discrimination across a wide range matters, and a two-hue diverging ramp about zero
for error, where the question is only which direction and how much. Each was
checked numerically for lightness progression, for contrast against the basemap it
sits on, and for separation under simulated colour-vision deficiency; one ramp was
re-stepped after measurement showed three adjacent colours that were
indistinguishable even to full-colour vision. When a light theme was added, the
error ramp needed its own steps rather than reuse, because a ramp that brightens
toward its extremes on a dark background inverts on a light one. Measurement
showed the largest errors dropping to 1.6:1 contrast while the zero-error midpoint
became the most prominent thing on the map.

## Delivery

Delivery follows from the same realization that reshaped the system design. This is
a catalog of pre-computed forecasts, not a live service, so it is static files on a
CDN with no backend, no inference server, and no database needed. A full seven-day
forecast is 28 frames on a 112×256 grid, which as 16-bit integers at 0.1 °C
precision is about 1.6 MB, small enough that the no server option was viable. The contract between the offline generator and the browser is written down as a versioned schema with grid orientation, byte layout, and value scaling in which frames have verifying observations. So both sides check against a document rather than against each other, and a future live pipeline can append to the same catalog without the interface changing at all. Two lessons came
out of building it. The first is that the development server is not the artifact.
A bug that left the map completely blank appeared only in the production build,
because a dependency resolved a worker file differently once bundled, and it
surfaced with no console error beyond a single misleading MIME-type line. Nothing
is verified until the built output is verified. The second is that a visualization
should be constrained to the data that backs it. The map is bounded to the model's
own domain, initally a user could zoom anywhere they wanted, and it would leave empty space outisde the data domain. Now no amount of zooming or panning can reach empty space where the
model has no data to present.
