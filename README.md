
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Extended two-way fixed effects (ETWFE)

<!-- badges: start -->

[![R-CMD-check](https://github.com/grantmcdermott/etwfe/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/grantmcdermott/etwfe/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

The goal of **etwfe** is to estimate extended (Mundlak) two-way fixed
effects *a la* [Wooldridge
(2021)](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3906345).
Briefly, Wooldridge proposes a set of saturated interaction effects to
overcome the potential bias problems that arise from using vanilla TWFE
in difference-in-differences designs. The Wooldridge solution is
intuitive and elegant, but is rather tedious and error prone to code up
manually. This package aims to simplify the process by providing
convenience functions that do the work for you. **etwfe** thus provides
an R equivalent of the
[`JWDID`](https://ideas.repec.org/c/boc/bocode/s459114.html) Stata
module and, indeed, shares some of the core design elements (albeit with
some internal differences).

*Note:* While I’ve tested **ewtfe** against common use cases, the
package is still under early development and should be considered
experimental. I plan (hope) to add some more features and the
documentation could also be improved. You can help by identifying any
bugs and filing issues.

## Installation

You can install the development version of **etwfe** from
[GitHub](https://github.com/):

``` r
# install.packages("remotes")
remotes::install_github("grantmcdermott/etwfe")
```

## Examples

To demonstrate the core functionality of **etwfe**, I’ll follow the lead
of `JWDID` by using an example dataset from the **did** package.

``` r
# install.packages("did")
data("mpdta", package = "did")
head(mpdta)
#>     year countyreal     lpop     lemp first.treat treat
#> 866 2003       8001 5.896761 8.461469        2007     1
#> 841 2004       8001 5.896761 8.336870        2007     1
#> 842 2005       8001 5.896761 8.340217        2007     1
#> 819 2006       8001 5.896761 8.378161        2007     1
#> 827 2007       8001 5.896761 8.487352        2007     1
#> 937 2003       8019 2.232377 4.997212        2007     1
```

Now let’s see a simple example. Note that the `gref` argument will be
unnecessary in most cases. But we invoke it here explicitly, since the
“never-treated” group in the `mpdta` dataset takes on an unusual value
(here: 0).

``` r
library(etwfe)

mod = 
  etwfe(
    fml  = lemp ~ lpop,           # outcome ~ controls
    tvar = year,                  # time variable
    gvar = first.treat, gref = 0, # group variable (with bespoke ref. level)
    data = mpdta,                 # dataset
    vcov = ~countyreal            # vcov adjustment (here: clustered)
    )
mod
#> OLS estimation, Dep. Var.: lemp
#> Observations: 2,500 
#> Fixed-effects: first.treat: 4,  year: 5
#> Varying slopes: lpop (first.treat: 4),  lpop (year: 5)
#> Standard-errors: Clustered (countyreal) 
#>                                               Estimate Std. Error   t value
#> .Dtreat:first.treat::2004:year::2004         -0.021248   0.021728 -0.977890
#> .Dtreat:first.treat::2004:year::2005         -0.081850   0.027375 -2.989963
#> .Dtreat:first.treat::2004:year::2006         -0.137870   0.030795 -4.477097
#> .Dtreat:first.treat::2004:year::2007         -0.109539   0.032322 -3.389024
#> .Dtreat:first.treat::2006:year::2006          0.002537   0.018883  0.134344
#> .Dtreat:first.treat::2006:year::2007         -0.045093   0.021987 -2.050907
#> .Dtreat:first.treat::2007:year::2007         -0.045955   0.017975 -2.556568
#> .Dtreat:first.treat::2004:year::2004:lpop_dm  0.004628   0.017584  0.263184
#>                                                Pr(>|t|)    
#> .Dtreat:first.treat::2004:year::2004         3.2860e-01    
#> .Dtreat:first.treat::2004:year::2005         2.9279e-03 ** 
#> .Dtreat:first.treat::2004:year::2006         9.3851e-06 ***
#> .Dtreat:first.treat::2004:year::2007         7.5694e-04 ***
#> .Dtreat:first.treat::2006:year::2006         8.9318e-01    
#> .Dtreat:first.treat::2006:year::2007         4.0798e-02 *  
#> .Dtreat:first.treat::2007:year::2007         1.0866e-02 *  
#> .Dtreat:first.treat::2004:year::2004:lpop_dm 7.9252e-01    
#> ... 6 coefficients remaining (display them with summary() or use argument n)
#> ... 10 variables were removed because of collinearity (.Dtreat:first.treat::2006:year::2004, .Dtreat:first.treat::2006:year::2005 and 8 others [full set in $collin.var])
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> RMSE: 0.537131     Adj. R2: 0.87167 
#>                  Within R2: 8.449e-4
```

As you can see, the key `etwfe()` function is effectively a wrapper
around `fixest::feols()`. (Although, non-linear models are also
supported via the `family` argument.) The resulting object is thus fully
compatible with other **fixest** methods and functions like `etable()`.

``` r
fixest::etable(mod, signif.code = NA)
#>                                                                   mod
#> Dependent Var.:                                                  lemp
#>                                                                      
#> .Dtreat x first.treat = 2004 x year = 2004           -0.0213 (0.0217)
#> .Dtreat x first.treat = 2004 x year = 2005           -0.0819 (0.0274)
#> .Dtreat x first.treat = 2004 x year = 2006           -0.1379 (0.0308)
#> .Dtreat x first.treat = 2004 x year = 2007           -0.1095 (0.0323)
#> .Dtreat x first.treat = 2006 x year = 2006            0.0025 (0.0189)
#> .Dtreat x first.treat = 2006 x year = 2007           -0.0451 (0.0220)
#> .Dtreat x first.treat = 2007 x year = 2007           -0.0459 (0.0180)
#> .Dtreat x lpop_dm x first.treat = 2004 x year = 2004  0.0046 (0.0176)
#> .Dtreat x lpop_dm x first.treat = 2004 x year = 2005  0.0251 (0.0179)
#> .Dtreat x lpop_dm x first.treat = 2004 x year = 2006  0.0507 (0.0211)
#> .Dtreat x lpop_dm x first.treat = 2004 x year = 2007  0.0112 (0.0266)
#> .Dtreat x lpop_dm x first.treat = 2006 x year = 2006  0.0389 (0.0165)
#> .Dtreat x lpop_dm x first.treat = 2006 x year = 2007  0.0381 (0.0225)
#> .Dtreat x lpop_dm x first.treat = 2007 x year = 2007 -0.0198 (0.0162)
#> Fixed-Effects:                                       ----------------
#> first.treat                                                       Yes
#> year                                                              Yes
#> Varying Slopes:                                      ----------------
#> lpop (first.treat)                                                Yes
#> lpop (year)                                                       Yes
#> ________________________________________             ________________
#> S.E.: Clustered                                        by: countyreal
#> Observations                                                    2,500
#> R2                                                            0.87321
#> Within R2                                                     0.00084
```

While everyone likes a nice regression table, the raw coefficients from
an `etwfe()` estimation are not necessarily meaningful in of themselves.
Instead, we probably want to aggregate them along some dimension of
interest (e.g., an event study). A natural way to perform these
aggregations is by calculating marginal effects. The **etwfe** package
provides another convenience function for doing this, `emfx()`, which is
itself a thin(ish) wrapper around `marginaleffects::marginaleffects()`

``` r
# Other type options incl. "simple" (default), "calendar", and "group"
emfx(mod, type = "event")
#>      Term    Contrast event   Effect Std. Error z value   Pr(>|z|)    2.5 %
#> 1 .Dtreat mean(dY/dX)     0 -0.03321    0.01337  -2.484 0.01297951 -0.05941
#> 2 .Dtreat mean(dY/dX)     1 -0.05735    0.01715  -3.343 0.00082830 -0.09097
#> 3 .Dtreat mean(dY/dX)     2 -0.13787    0.03079  -4.477 7.5665e-06 -0.19823
#> 4 .Dtreat mean(dY/dX)     3 -0.10954    0.03232  -3.389 0.00070142 -0.17289
#>     97.5 %
#> 1 -0.00701
#> 2 -0.02373
#> 3 -0.07751
#> 4 -0.04619
#> 
#> Model type:  etwfe 
#> Prediction type:  response
```

## Acknowledgements

- [Jeffrey Wooldridge](https://twitter.com/jmwooldridge) for the
  [underlying
  theory](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3906345)
  that drives this package.
- [Laurent Bergé](https://twitter.com/lrberge)
  ([**fixest**](https://lrberge.github.io/fixest/)) and [Vincent
  Arel-Bundock](https://twitter.com/VincentAB)
  ([**marginaleffects**](https://vincentarelbundock.github.io/marginaleffects))
  for maintaining the two wonderful R packages that do most of the heavy
  lifting under the hood here.
- [Fernando Rios-Avila](https://twitter.com/friosavila) for the
  [`JWDID`](https://ideas.repec.org/c/boc/bocode/s459114.html) Stata
  module, which has provided a welcome foil for checking results and
  whose elegant design helped inform my own choices for this R
  equivalent.
