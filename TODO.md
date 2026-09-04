# TODO — Code migration

The R and C code is currently only available as screenshots in the report.
To restore this repository to a fully executable state, the following files should be added.

## Priority 1 — sampling of rremb

- [ ] `R/rremb_rejection.R` — uniform-envelope rejection sampler with dominance constant
- [ ] `R/rremb_inversion.R` — analytic F⁻¹ with piecewise simplification
- [ ] `R/rremb_benchmark.R` — timing and histogram comparison

## Priority 2 — weather driver

- [ ] `R/weather.R` — vectorised Markov-chain weather simulator over N years
- [ ] `R/accident_probability.R` — weather-conditional accident probability

## Priority 3 — conditional expected excess

- [ ] `R/msa.R` — direct Monte-Carlo estimator of *m(s) = E(R − s | R > s)*
- [ ] `R/msb.R` — alternative estimator with conditional reparameterisation
- [ ] `R/msab_compare.R` — variance and timing comparison

## Priority 4 — C port

- [ ] `src/msb.c` — full C implementation of the msb estimator (5 parts of the report)
- [ ] `R/msb_wrapper.R` — R wrapper calling msb.c via .Call
- [ ] `Makefile` (or `NAMESPACE` + package skeleton) to build the shared library

## Priority 5 — analysis

- [ ] `R/sensitivity.R` — sensitivity to rain-accident probability, x₀, η
- [ ] `examples/` — small runnable script reproducing figures 26–30 of the report

## Reproducibility

- [ ] `renv.lock` for R dependencies
- [ ] `README` build instructions for the C shared library on Windows and Linux
