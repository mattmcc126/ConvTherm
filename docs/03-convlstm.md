# ConvLSTM

## Architecture

A ConvLSTM replaces the stack of raw input frames with a persistent hidden state
carried across steps. Instead of re-reading history, the model maintains a learned,
filtered summary of the atmosphere's trajectory and updates it in steps. Two
properties of this architecture attack the baseline's failure directly. The cell's forget gate is a sigmoid function, and therefore strictly between zero and one, so the step-to-step gain on
the memory path is explicitly bounded below one wherever the network learns it
should be, whereas the U-Net had no analogue, because its state was the raw field and the
recursion on it was unconstrained. Recurrence makes multi-step training
tractable, which means the loss can span several steps, the gradient can carry
information about how errors compound, and the optimizer can trade slightly worse
one-step accuracy for a smaller amplification factor. That trade was structurally
unavailable with the U-Net. The design is a hybrid rather than a pure ConvLSTM, and both
rejected alternatives were rejected for the stated reasons. Full-resolution recurrence
is prohibitive to backpropagate through time and misallocates capacity, since
six-hour evolution is dominated by synoptic-scale motion that lives at coarse
resolution. Dropping skip connections would force every fine spatial detail through
a coarse bottleneck, discarding exactly the terrain-locked gradients, coastlines,
and urban signatures the convolutional baseline demonstrably handled well. The
final shape keeps a shared-weight convolutional encoder per frame, places recurrent
memory at the two coarsest scales where the physics is, and reconstructs full
resolution through a mirror decoder with skip connections and a residual head — 36
million parameters, against 73 million for the baseline it replaced.

## Training and testing

The model warms up on several observed frames to charge its memory before producing
any forecast, then rolls forward autoregressively. Every frame the encoder sees carries the
clock of its own contents, not of the target being predicted. The alternative
convention pairs a state with a timestamp six hours ahead of it, which shifts the
entire forecast by one step and presents as a mediocre model rather than as a
plumbing bug. Validation always rolled out to a fixed number of steps at every
training stage, even stages trained on fewer, so the curves from different stages
remain directly comparable. The quantity actually monitored during multi-step
training was the gap between one-step and multi-step error, since that gap is the
observable proxy for known error amplification, a stage succeeds by narrowing it, even if
one-step error drifts slightly upward, and monitoring one-step error instead would
have checkpointed the wrong epoch every time. Testing reused the baseline's metric
code and held-out years unchanged, so the comparison is as identical as possible rather than
two models measured with two different metric sets.

## Rollout fine-tuning

Training ran as a curriculum, first on single six-hour steps, then fine-tuned in
stages on progressively longer rollouts of 2, 4, 8, and 12 steps, each resuming
from the previous stage's best checkpoint at a much lower learning rate. The extra stages are meant to bend the amplification factor, not to relearn the one-step map. The outcome contained the most useful negative result in the project: more rollout in training was not monotonically better. The 8-step stage produced the best model, and extending to 12 steps made forecasts measurably worse at two-to-five-day leads. That was only visible because every stage was
evaluated independently instead of assuming the final one was best. The finished
model beats the U-Net at every lead about 1.4% better at 6 hours rising to 9.0%
at 168 hours with half the parameters, and it roughly halves the seven-day
deficit against climatology, from −0.26 to −0.14. It does not eliminate it, and it
was never going to. At around 20 m/s, synoptic systems traverse the chosen domain in
roughly 72 hours, so by day three essentially every air parcel over the map entered
from outside it, where the model has no information at all. That is a structural
limit of a limited-area model without boundary conditions, and it is why the next step would be a larger domain rather than a larger network. One final
evaluation lesson was scoring on a single held-out year was misleading by around 20%
at long leads, because one of the three test years happened to contain unusually
severe winter weather. Reporting results correctly meant pooling all three rather
than quoting the year that fooled the model.
