
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mread.yaml

<!-- badges: start -->

<!-- badges: end -->

mread.yaml provides alternate model specification formats, primarily
based on yaml.

## Installation

``` r
# install.packages("devtools")
devtools::install_github("kylebaron/mread.yaml")
```

## Example

### Setup

``` r
library(mread.yaml)
library(dplyr)
library(mrgsolve)
```

Read some input
code

``` r
examp <- system.file("examples", "two_cmt_model.cpp", package = "mread.yaml")
```

This looks identical to the regular model specification file, but rather
than writing differential equations, we write the model in reaction
notation.

``` yaml
[ prob ]
This is a simple pk model; going to demonstrate how we can specify reactions
as fluxes and put them together as reactions.

[ cmt ] gut cent periph

[ param ] ka = 1.2, cl = 1, v1 = 10, v2 = 40, q = 5

[ main ]
double k10 = cl/v1;
double k12 = q/v2;
double k21 = q/v2;

[ ode_assignments ]
double conc = cent/v1;

[ reactions ]
- species: gut --> cent
  form: ka * gut
- species: cent --> NULL
  form: cl * conc
- species: cent <--> periph
  form: k12 * cent - k21 * periph

[ table ] capture cp = cent/v1;

[capture] k10
```

In this simple example, we write the model as a series of fluxes:

  - absorption from the gut to the central compartment (1)
  - elimination (2)
  - transfer between the central and peripheral compartments (3)

mread.yaml provides a function to read and parse this type of model
specification file and return a model object just like
\[mrgsolve::mread\].

``` r
mod <- mread_rxn(examp, delta = 0.1, end = 48)
```

Once the model is loaded, it functions just like you had written it
using ODEs

``` r
mod
. 
. 
. ------------  source: two_cmt_model.cpp  ------------
. 
.   project: /private/var/fol.../T/RtmpHMZmRC
.   shared object: two_cmt_model.cpp-so-2f0cc2f54d6 
. 
.   time:          start: 0 end: 48 delta: 0.1
.                  add: <none>
. 
.   compartments:  gut cent periph [3]
.   parameters:    ka cl v1 v2 q [5]
.   captures:      k10 cp [2]
.   omega:         0x0 
.   sigma:         0x0 
. 
.   solver:        atol: 1e-08 rtol: 1e-08 maxsteps: 20k
. ------------------------------------------------------
```

``` r
mrgsim(mod, ev(amt = 100)) %>% plot
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

In fact, all `mread_rxn()` does, is parse the reactions and write that
math as ODEs

``` c
[ PROB ]
This is a simple pk model; going to demonstrate how we can specify reactions
as fluxes and put them together as reactions.
 
[ CMT ]
gut cent periph
 
[ PARAM ]
ka = 1.2, cl = 1, v1 = 10, v2 = 40, q = 5
 
[ MAIN ]
double k10 = cl/v1;
double k12 = q/v2;
double k21 = q/v2;
 
[ TABLE ]
capture cp = cent/v1;
 
[ CAPTURE ]
k10
 
[ ODE ]
double conc = cent/v1;

double J01 = ka * gut;
double J02 = cl * conc;
double J03 = k12 * cent - k21 * periph;
 
dxdt_gut    = -J01;
dxdt_cent   =  J01 -J02 -J03;
dxdt_periph =  J03;

//-------------------------------

 
```