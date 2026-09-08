# International Air Passengers — Forecast Report

## The recommendation
Ship AutoARIMA over the seasonal-naive floor: it roughly halves the
forecast error (MASE 0.65 vs. 1.31), validated across 8 rolling-origin
cross-validation windows, not a single lucky holdout. The improvement
holds consistently, not just on average — AutoARIMA's worst window
(MASE 1.59) still beats the floor's worst window (1.96). RMSSE (0.68
vs 1.25) and CRPS (0.034 vs 0.062) tell the same story.

<img width="694" height="364" alt="MASE comparison" src="https://github.com/user-attachments/assets/cc89382c-c4e9-4f83-810e-59ad875162b9" />

## The intervals
Neither model's 80% intervals are well-calibrated: SeasonalNaive covers
only 51% of actuals, AutoARIMA 70% — both overconfident, though
AutoARIMA is meaningfully closer to honest. At a stated 80% confidence
level, an interval that only covers the truth half the time is not a
band a manager should trust at face value. This is a real limitation
worth flagging, not just a footnote.

## The residuals
Ljung-Box on AutoARIMA's cross-validated residuals still rejects white
noise at both lag 12 (p ≈ 6e-26) and lag 24 (p ≈ 1e-37) — some structure
remains uncaptured. However, the test statistic dropped substantially
versus the floor (150 vs 234 at lag 12), consistent with AutoARIMA
absorbing most, not all, of the pattern.

Two candidate explanations: (1) the series showed growing seasonal
amplitude in the STL decomposition — a signature of multiplicative
seasonality — and this model was fit on the raw scale rather than a
log transform, which may leave some of that structure unmodeled; (2)
stitching 8 rolling-origin windows into one residual series for this
test can itself introduce artificial structure at the window seams,
which Ljung-Box (built for one continuous series) may partly be
picking up rather than a true model deficiency.

## One change
The next thing I'd try is fitting on a log scale instead of the raw
passenger counts. The decomposition showed the seasonal swings and the
residual noise both growing right alongside the trend — the textbook sign
of multiplicative seasonality, not additive. AutoARIMA was fit on the raw
scale, and its leftover residuals still show significant structure at
both lag 12 and lag 24, some of which is likely this unmodeled growth
in seasonal amplitude.

Taking the log of the series before fitting turns that multiplicative
pattern into an additive one, which ARIMA-family models are built to
handle cleanly. I'd expect this to do two things: further reduce the
leftover structure in the residuals (moving Ljung-Box's p-value closer
to "looks like noise"), and produce forecast intervals that widen more
realistically in the later, higher-volatility years — directly helping
the coverage problem above, since a chunk of that overconfidence
likely comes from treating a growing-variance series as if its swings
were constant.

## Extra credit: dynamic regression with an external driver
As an experiment, I added Riyadh's monthly mean temperature (via
Open-Meteo's historical archive) as an exogenous regressor in AutoARIMA,
run through the same 8-window harness as the other models.

Result: essentially no improvement. MASE (0.651 vs 0.647) and RMSSE
(0.685 vs 0.683) are statistically indistinguishable from plain
AutoARIMA; CRPS is identical. Coverage improved slightly (0.719 vs
0.698). This is the expected outcome, not a bug: Riyadh has no real
causal link to this dataset's international passenger totals, so the
model correctly found little signal to exploit. This illustrates that
an external driver only helps forecasting when it has a genuine
mechanistic connection to the target — an important, if slightly
negative, result in its own right.

<img width="754" height="364" alt="Three-way MASE comparison" src="https://github.com/user-attachments/assets/c24e8686-1884-4eb7-ac2b-af0d886373cc" />


