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

