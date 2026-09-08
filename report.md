   ## Report notes (draft — will become report.md)
      ### Data
   The series covers 144 consecutive months (Jan 1949–Dec 1960) with no 
   missing values, sourced from the classic Box & Jenkins airline dataset.

### First read
Passenger numbers climbed steadily from 1949 to 1960, nearly tripling. 
Every year has the same summer peak and winter dip, but as the years 
go on those peaks and dips get more dramatic — not because travel got 
more seasonal, but because there's simply more travel overall for the 
seasons to swing around.
<img width="1002" height="562" alt="stat1" src="https://github.com/user-attachments/assets/4ee9662b-1a1b-403b-bda9-775ecdf98f0a" />

### Decomposition
STL decomposition confirms the visual read: a smooth upward trend and a 
yearly seasonal pattern. Both the seasonal component's amplitude and the 
residual's spread grow over time rather than staying constant — evidence 
that the series is multiplicative, and a candidate reason to model on a 
log scale.
<img width="993" height="540" alt="stat2" src="https://github.com/user-attachments/assets/e93f4a78-81b2-43fb-b806-76e0b5beadb4" />

### Trend and seasonal strength
Trend strength: 1.00, seasonal strength: 0.98 — both components explain 
almost all of the variation in the series, leaving very little unexplained 
noise. This means the benchmark floor should be a seasonal-naive model 
(not a plain naive one), since the strong seasonality would make a 
non-seasonal floor trivially easy to beat.

### Autocorrelation
The ACF shows slow, steady decay across all 24 lags rather than a sharp 
cutoff — the fingerprint of a trending, non-stationary series. Small 
bumps at lag 12 and lag 24 confirm the yearly seasonal pattern is real. 
Together this points to differencing (and seasonal differencing) being 
necessary before fitting an ARIMA-family model.

<img width="584" height="364" alt="stat3" src="https://github.com/user-attachments/assets/4feba5a0-633e-4a10-870c-a74ae0068b0a" />

### The floor
The chart makes it visible: Seasonal Naive forecasts 1961 and 1962 as 
identical repeats of the same yearly shape, with no continued climb. 
Given the series' trend strength of 1.00, the real future is expected 
to keep rising above this flat, repeating forecast — this is the gap 
our model needs to close to earn its place over the floor.
<img width="1112" height="518" alt="stat5" src="https://github.com/user-attachments/assets/62748d62-af7a-4eab-a39e-ab6675df4d31" />


### Floor uncertainty
The 80%/95% bands widen and narrow with the season itself — wider 
around the volatile summer peaks, narrower around the calmer winter 
troughs — and grow slightly wider overall in the second forecast year 
than the first. The floor is honestly representing its own uncertainty, 
even though its point forecasts don't account for trend.

<img width="1002" height="452" alt="stat6" src="https://github.com/user-attachments/assets/d2fcc638-a7ea-4943-a511-3427334cc144" />


### What the floor leaves on the table
Ljung-Box on the floor's residuals (y_t - y_t-12) rejects white noise 
overwhelmingly at both lag 12 (p ≈ 2e-43) and lag 24 (p ≈ 2e-44). This 
isn't randomness the year-over-year differences are consistently 
positive and growing, which is exactly the trend the floor can't see. 
Any model that adds a trend component on top of seasonality (SARIMA, 
Holt-Winters/ETS) should be able to capture this structure and beat 
the floor.

### AutoGluon leaderboard 
Quick AutoGluon run (2-min time limit) shows AutoARIMA and its 
WeightedEnsemble beating SeasonalNaive on score_val (~-1.54 vs -1.66), 
suggesting a trend-aware model likely beats the floor. This is not 
a validated harness result — only the rolling-origin cross-validation 
numbers count for the report.


### Reading the AutoGluon leaderboard
score_val is negative by AutoGluon convention (closer to zero = better). 
No windows column is shown, meaning this ranking likely reflects a 
single 12-month holdout too fragile to trust as a final result, only 
as a rough shortlist. That said, it agrees with every earlier finding: 
trend-aware models (AutoARIMA) beat the seasonal-only floor, and plain 
Naive performs worst. This motivates testing an ARIMA-family model 
properly through the rolling-origin harness next.


### The harness
Built the rolling-origin cross-validation: 8 independent origins 
(Dec 1952 through Dec 1959, spaced 12 months apart), each forecasting 
12 months ahead against real held-out actuals — 96 forecast/actual 
pairs total for the floor model. Already visible in the raw output: 
the floor consistently under-forecasts, consistent with the trend 
the floor can't see (per section 2.1's Ljung-Box result).

### The recommendation
Ship AutoARIMA over the seasonal-naive floor: it roughly halves the 
forecast error (MASE 0.65 vs. 1.31), validated across 8 rolling-origin 
cross-validation windows, not a single lucky holdout. The improvement 
holds consistently, not just on average — AutoARIMA's worst window 
(MASE 1.59) still beats the floor's worst window (1.96). RMSSE (0.68 
vs 1.25) and CRPS (0.034 vs 0.062) tell the same story.

<img width="694" height="364" alt="stat7" src="https://github.com/user-attachments/assets/cc89382c-c4e9-4f83-810e-59ad875162b9" />


### The intervals
Neither model's 80% intervals are well-calibrated: SeasonalNaive covers 
only 51% of actuals, AutoARIMA 70% — both overconfident, though 
AutoARIMA is meaningfully closer to honest. This is a real limitation 
worth flagging, not just a footnote.
