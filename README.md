# EllipticalSliceSampling.jl

Julia implementation of elliptical slice sampling.

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)
[![Build Status](https://travis-ci.com/devmotion/EllipticalSliceSampling.jl.svg?branch=master)](https://travis-ci.com/devmotion/EllipticalSliceSampling.jl)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/devmotion/EllipticalSliceSampling.jl?svg=true)](https://ci.appveyor.com/project/devmotion/EllipticalSliceSampling-jl)
[![Codecov](https://codecov.io/gh/devmotion/EllipticalSliceSampling.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/devmotion/EllipticalSliceSampling.jl)
[![Coveralls](https://coveralls.io/repos/github/devmotion/EllipticalSliceSampling.jl/badge.svg?branch=master)](https://coveralls.io/github/devmotion/EllipticalSliceSampling.jl?branch=master)

## Overview

This package implements elliptical slice sampling in the Julia language, as described in
[Murray, Adams & MacKay (2010)](http://proceedings.mlr.press/v9/murray10a/murray10a.pdf).

Elliptical slice sampling is a "Markov chain Monte Carlo algorithm for performing
inference in models with multivariate Gaussian priors" (Murray, Adams & MacKay (2010)).

Without loss of generality, the originally described algorithm assumes that the Gaussian
prior has zero mean. For convenience we allow the user to specify arbitrary Gaussian
priors with non-zero means and handle the change of variables internally.

## Usage

Probably most users would like to use the exported function
```julia
ESS_mcmc([rng::AbstracRNG, ]prior, loglikelihood, N::Int[; burnin::Int = 0])
```
which returns a vector of `N` samples for approximating the posterior of
a model with a Gaussian prior that allows sampling from the `prior` and
evaluation of the log likelihood `loglikelihood`. The burn-in phase with
`burnin` samples is discarded.

If you want to have more control about the sampling procedure (e.g., if you
only want to save a subset of samples or want to use another stopping
criterion), the function
```julia
ESS_mcmc_sampler([rng::AbstractRNG, ]prior, loglikelihood)
```
gives you access to an iterator from which you can generate an unlimited
number of samples.

### Prior

You may specify Gaussian priors with arbitrary means. EllipticalSliceSampling.jl
provides first-class support for the scalar and multivariate normal distributions
in [Distributions.jl](https://github.com/JuliaStats/Distributions.jl). For
instance, if the prior distribution is a standard normal distribution, you can
choose
```julia
prior = Normal()
```

However, custom Gaussian priors are supported as well. For instance, if you want to
use a custom distribution type `GaussianPrior`, the following methods should be
implemented:
```julia
# state that the distribution is actually Gaussian
EllipticalSliceSampling.isnormal(::Type{<:GaussianPrior}) = true

# define the mean of the distribution
Statistics.mean(dist::GaussianPrior) = ...

# define how to sample from the distribution
# only one of the following methods is needed:
# - if the samples are immutable (e.g., numbers or static arrays) only
#   `rand(rng, dist)` should be implemented
# - otherwise only `rand!(rng, dist, sample)` is required
Base.rand(rng::AbstractRNG, dist::GaussianPrior) = ...
Random.rand!(rng::AbstractRNG, dist::GaussianPrior, sample) = ...

# specify the type of a sample from the distribution
Base.eltype(::Type{<:GaussianPrior}) = ...

# in the case of mutable samples, specify the array size of the samples
Base.size(dist::GaussianPrior) = ...
```

### Log likelihood

In addition to the prior, you have to specify a Julia implementation of
the log likelihood function. Here the predefined log densities and log
likelihood functions in
[Distributions.jl](https://github.com/JuliaStats/Distributions.jl) might
be useful.

### Progress monitor

If you use a package such as [Juno](https://junolab.org/) or
[ConsoleProgressMonitor.jl](https://github.com/tkf/ConsoleProgressMonitor.jl) that supports
progress logs created by the
[ProgressLogging.jl](https://github.com/JunoLab/ProgressLogging.jl) API, then you can
monitor the progress of the sampling algorithm.

## Bibliography

Murray, I., Adams, R. & MacKay, D.. (2010). Elliptical slice sampling. Proceedings of Machine Learning Research, 9:541-548.
