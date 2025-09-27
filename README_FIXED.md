# mHMMbayes - Fixed Version with NA Handling for Categorical Data

This is a fork of the [mHMMbayes package](https://github.com/emmekeaarts/mHMMbayes) with a fix for handling missing data (NAs) in categorical input variables.

## Problem Solved

The original mHMMbayes package throws an error when categorical data contains NA values:
```
Error: For categorical input variables, the values should be integers ranging from 1 to number of observed categories specified for each of the dependent variables in q_emiss
```

This fix allows the package to handle NAs in categorical data, making it consistent with how continuous data is already handled.

## Installation

You can install this fixed version directly from GitHub:

```r
# Install devtools if not already installed
if (!require("devtools")) {
  install.packages("devtools")
}

# Install the fixed version
devtools::install_github("andreifoldes/mHMMbayes-fixed")
```

## What Changed

The validation check in `R/mHMM.R` was modified to properly handle NAs:
- Uses `na.rm = TRUE` for min/max calculations
- Filters out NAs when checking unique values per category
- Allows the model to proceed with missing data

## How NAs are Handled

The model handles NAs intelligently:
- **During state inference**: Ignores emission probabilities at NA time points, relies only on transition probabilities
- **During parameter estimation**: Excludes NAs from parameter updates (only uses observed data)
- **Time series structure**: Preserved - doesn't skip or remove NA time points
- **No imputation**: NAs are never filled in with guessed values

## Example Usage

```r
library(mHMMbayes)

# Load example data
data(nonverbal)

# Introduce some missing data
set.seed(42)
nonverbal_na <- nonverbal
n_missing <- 100
missing_indices <- sample(1:nrow(nonverbal_na), n_missing)
nonverbal_na[missing_indices, 2] <- NA  # Add NAs to first dependent variable

# Model specifications
m <- 2
n_dep <- 4
q_emiss <- c(3, 2, 3, 2)

# Starting values
start_tm <- diag(.8, m)
start_tm[lower.tri(start_tm) | upper.tri(start_tm)] <- .2
start_em <- list(
  matrix(c(0.05, 0.90, 0.05, 0.90, 0.05, 0.05), byrow = TRUE, nrow = m, ncol = q_emiss[1]),
  matrix(c(0.1, 0.9, 0.1, 0.9), byrow = TRUE, nrow = m, ncol = q_emiss[2]),
  matrix(c(0.90, 0.05, 0.05, 0.05, 0.90, 0.05), byrow = TRUE, nrow = m, ncol = q_emiss[3]),
  matrix(c(0.1, 0.9, 0.1, 0.9), byrow = TRUE, nrow = m, ncol = q_emiss[4])
)

# This now works with NAs!
out_na <- mHMM(s_data = nonverbal_na,
               gen = list(m = m, n_dep = n_dep, q_emiss = q_emiss),
               start_val = c(list(start_tm), start_em),
               mcmc = list(J = 1000, burn_in = 200))

# View results
summary(out_na)
```

## Citation

If you use this package, please cite the original mHMMbayes package:

Aarts, E., Emmeke (2023). mHMMbayes: Multilevel Hidden Markov Models Using Bayesian Estimation. R package version 1.1.0. https://CRAN.R-project.org/package=mHMMbayes

## Original Package

For the original package and documentation, see: https://github.com/emmekeaarts/mHMMbayes

## Fix Details

- **Issue**: Validation check failed when categorical data contained NAs
- **Solution**: Modified validation to properly handle NAs using `na.rm=TRUE` and filtering
- **File changed**: `R/mHMM.R` (lines 579-581)
- **Commit**: d458700