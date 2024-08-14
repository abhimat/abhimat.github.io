---
layout: default
nav_cat: software
title: Software
---

Examples of public software that I have led the development of or contributed to.

## Lead contributor

### Phitter

[Phitter](https://abhimat.net/phitter/index.html) is an open-source python package to simulate observables from stellar binary systems and to fit them to observed data. Observables that can be calculated and fit with Phitter include photometry (i.e., observed fluxes) and line-of-sight radial velocities (RVs).

Modeling of binary systems and calculation of observables is primarily handled with PHOEBE. When computing flux from model binaries, synthetic photometry for stars is derived for a wide range of telescope and passbands using SPISEA. Parameters for the binary system’s stellar components can be derived via interpolation of model stellar tracks (Phitter currently implements MIST). Otherwise, arbitrary stellar parameters for one or both stars can also be specified.

Fitting of observables to binary models is conducted with the use of MCMC sampling code. We provide support for sampling with nested sampling codes like UltraNest, MultiNest (via PyMultiNest), or dynesty. Phitter can also be used with non-nested sampling MCMC codes like emcee. Examples to demonstrate how to set up a fitter are provided for UltraNest, dynesty, and emcee.

[![DOI](https://zenodo.org/badge/170761219.svg)](https://zenodo.org/doi/10.5281/zenodo.8370775)

* [Documentation](https://abhimat.net/phitter/index.html)
* Git Repository: [Github](https://github.com/abhimat/phitter/)
* Citation: [zenodo](https://zenodo.org/doi/10.5281/zenodo.8370775)

### Analysis software for science papers

#### [Gautam et al. (2024)](https://doi.org/10.3847/1538-4357/aaf103)
* [`binary_fraction`](https://github.com/abhimat/binary_fraction): Python library used for the majority of the analysis, including the computation of significantly periodic signals, and determination of a stellar binary fraction estimate from observations.
* [gatspy](https://github.com/abhimat/gatspy): Fork of [gatspy](http://www.astroml.org/gatspy/) to implement a trended, multi-band Lomb-Scargle periodicity search. Long-term trends can be modeled with polynomials, and the periodicity search is performed using the framework described by [VanderPlas & Ivezić (2015)](https://ui.adsabs.harvard.edu/abs/2015ApJ...812...18V).


## Significant Contributions

### Keck AO Imaging (KAI) Data Release Pipeline
* [Documentation](https://keck-datareductionpipelines.github.io/KAI/)
* Git Repository: [Github](https://keck-datareductionpipelines.github.io/KAI/)
* Citation (v1.0.0): [zenodo](https://zenodo.org/records/6677744#.YrS6bS9h3UI)

### Stellar Population Interface for Stellar Evolution and Atmospheres (SPISEA)
* [Documentation](https://spisea.readthedocs.io/en/latest/)
* Git Repository: [Github](https://github.com/astropy/SPISEA)
* Citation: [Hosek et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020AJ....160..143H/abstract)