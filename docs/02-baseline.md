# Baseline

## Baseline design

The baseline was a U-Net encoder–decoder architecture, which is a purely convolutional model that treats time as channel depth, stacking several recent frames into one tensor and
predicting the next atmospheric state in a single pass. Building it first was
a deliberate choice. It establishes a reproducible reference number, exercises the entire
data pipeline, metric code, and training loop before any architectural complexity
is introduced, and leaves a fallback if the more ambitious model fails to improve
on it. Several decisions were locked before writing the model, and most of them
outlived it. The forecast step was set at six hours with 28 steps to reach seven
days, rather than the hourly step originally planned beacuse hourly would have demanded
168 autoregressive steps and six times the data movement to resolve motion that is
dominated by synoptic-scale systems anyway. Each input frame carries static
channels alongside the physical variables, these include a land–sea mask, normalized latitude and longitude grids, and cyclical encodings of hour-of-day and day-of-year, so the
network can locate itself in space and in the diurnal and seasonal cycles. The
model predicts a change to the most recent observation rather than the next state
outright, which starts training near the identity map instead of near noise.
Finally, the lateral boundary problem that a regional model has no information
about weather flowing in from outside its domain was acknowledged in writing as a
known and explainable failure mode, to be measured rather than designed around.

## Baseline training and testing

Evaluation borrowed its instruments from the weather forecasting literature rather
than inventing them, which matters more than it sounds. Errors are
latitude weighted, because grid cells on a regular latitude–longitude grid cover
noticeably less ground toward the pole and an unweighted average quietly
overweights the north. Results are reported as anomaly correlation and as skill
scores against two references, not as a bare RMSE. Climatology, one of those
references, was computed from the training years only, using all years would have
leaked information about the test period into the very baseline the model is judged
against, and would have made the model look worse than it is while being
methodologically wrong. Metrics were also computed on an interior crop with the
boundary ring removed, so the effect of the domain edges could be separated from
the model's behaviour in the interior. The baseline after the training/testing cycle passed its gate. It reached a single-step validation RMSE of about 0.91 °C and beat both persistence and
climatology at 24, 48, and 72 hours, which was the criterion set before any of it
was built.

## What the results implied

The rollout profile was the real finding, and it was not good news. At five and
seven days the baseline scored worse than climatology skill scores of −0.17
and −0.26 against it. A forecast that decays gracefully toward the climatological average is behaving correctly, it is showing predictability has run out and falling back on the
long run mean. Going past climatology means the model is confidently wrong,
drifting away from the average rather than relaxing toward it. The diagnosis is
that forecast error has two independent sources how much error the model
introduces per step, and how much it amplifies error already present. Single-step
training penalizes only the first. It places no pressure whatsoever on the second,
because during training the model never sees its own output. A model can therefore
be excellent one step ahead and useless twenty-eight steps ahead, and single-step
validation gives no warning. The structural cause was equally clear: the U-Net has
no state. Its only representation of history is a stack of raw frames rebuilt from
scratch each step, and by late in a rollout most of those frames are its own
earlier predictions, re-read as though they were observations. The per-step error versus error amplification is what defined the requirement for the next architecture, and it is why the move to a recurrent model was an evidence-driven response to a measurement rather than a default upgrade to something fancier.
