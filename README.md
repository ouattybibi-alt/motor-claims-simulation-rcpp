# Motor Insurance Claims — Monte-Carlo Simulation and R/C++ Interfacing

Monte-Carlo estimation of the conditional expected excess loss for a policyholder portfolio, combining rejection and inversion sampling for the reimbursement distribution, a Markov-chain weather driver, and an R-to-C++ interface (via Rcpp) for the compute-intensive routines.

Academic project (Master 1 Actuariat, ISFA — Simulation), supervised by Prof. Alexis Bienvenüe. Co-authored with C. A. D. Kouamé and S. Ouattara.

## Content

Everything runs from a single R Markdown notebook:

- `R/motor_claims_simulation.Rmd` — full implementation: samplers, weather chain, both estimators (`msa`, `msb`), and their C++ port (`msb.c`)
- `projet_simu.pdf` — full write-up with derivations, C code listings and execution-time comparisons

## What the notebook does

### 1. Reimbursement random variable `rremb`

The reimbursement per accident follows a piecewise density with a discontinuity at α:

$$f(x) = (\alpha + |x - x_0|)^{-\eta}$$

Implemented by **inversion sampling** on the closed-form CDF F, with an analytic piecewise F⁻¹ that avoids the rejection-sampling waste on the peaked mode.

### 2. Weather Markov chain (`simulate_weather_vectorized`)

Three-state Markov chain (Sun / Cloudy / Rain) with transition parameters `(pSN, pNS, pNP, pPN)`. Vectorised sampler over a full year (365 days). Accident probability is state-dependent (higher under rain), which propagates into the loss distribution.

### 3. Conditional expected excess — `msa` and `msb`

Estimation of *m(s) = E(R − s | R > s)*, the expected excess loss above threshold s. Two estimators:

- **`msa`** — direct Monte-Carlo: simulate R, filter on R > s, average the excess. Simple but throws away every draw below s.
- **`msb`** — alternative estimator: perturbs the weather counts (nbS, nbN, nbP) by uniform noise bounded by the empirical inter-arrival standard deviations (Vn_S, Vn_N, Vn_P), keeping the total constant at 365 days. Every simulated year contributes.

Both estimators run on the same parameters and are compared on execution time.

### 4. R ↔ C++ interfacing (`msb.c`)

The `msb` estimator is ported to C++ via Rcpp `cppFunction`, in **two blocks** matching the report's split (Figures 17–21):

- **`msbc1`** replaces the R loop over weather positions (`which`, `diff` on the weather vector) with a native C++ implementation, returning a list of six elements — three `diff_indices` vectors and three integer counts (`nbr de S/N/P`) — that are then renamed R-side.
- **`msbc2`** ports the outer Monte-Carlo loop itself: for each simulation, sample the ε perturbations, build the perturbed weather, generate the accident matrix, call back into R for the `rremb` draws, and compute the excess.

The two `cppFunction` blocks include auxiliary C++ helpers (`which_indices`, `diff_indices`) via the `includes` argument.

### 5. Sensitivity and diagnostics

Sensitivity of `m(s)` to `x₀` and `η`, ggplot visualisations of the reimbursement density, and calibration checks (analytic density `dremb` versus the empirical histogram of `rremb` draws).

## Reference timings (from the report)

At `N = 10 000` motorbikes, `s = 4 000`, 1 000 simulations:

| Estimator | User time | System time | Elapsed |
|---|---:|---:|---:|
| `msb` (pure R) | — | — | ~375 s |
| `msb.c` (R + C++ via Rcpp) | — | — | ~297 s |

About a **20 % wall-clock reduction** on the alternative estimator by moving the two hot loops to C++.

## Requirements

```r
install.packages(c("Rcpp", "ggplot2"))
```

The Rcpp toolchain needs a working C++ compiler. On Windows, install **Rtools** matching your R version (rtools44 for R 4.4+).

## Quick start

```r
rmarkdown::render("R/motor_claims_simulation.Rmd")
```

or open the file in RStudio and knit it. The two `system.time(...)` calls at the end of the file reproduce the head-to-head R vs Rcpp timing.

## Note on `msbc2`

The report explicitly documents that the second C++ block (`msbc2`, porting the outer Monte-Carlo loop) was **retained despite showing no material improvement over the R version** — the variance and CI half-width came out slightly larger, and the wall-clock gain was negligible. It is kept here for completeness and traceability against the report; the meaningful speed-up comes from `msbc1`.

## Report

See `projet_simu.pdf` for the full methodology, derivations, C code listings and execution-time comparisons.
