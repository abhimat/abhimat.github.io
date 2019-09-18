---
layout: default
nav_cat: research
title: GC Old Binary
parent: Research
---

<center><p><a href="./Poster_Print.pdf">Download a copy of the poster</a></p></center>

## Photometric observations and<br>discovery of periodic variability

The discovery is based on photometric measurements of stars in the central 10 arcseconds of the Milky Way Galactic center (GC), corresponding to a projected distance of approximately 0.4 parsecs. These photometric measurements are derived from imaging data taken with Keck observatory NIRC2 laser guide star adaptive optics (AO) observations of the Galactic center, with observations spanning from 2006 to 2019. Observations were performed in near-infrared, in *K\'*-band (2.124 µm; 59 nights, 2006–2019) and in *H*-band (1.633 µm; 12 nights, 2017–2019).

The methodology to derive precise photometry from GC AO imaging data is described in [Gautam+ (2019)][Gautam+2019]. The paper presents the photometric dataset in *K\'* spanning 2006–2017 (45 nights). The photometry methodology includes selecting stable calibrator stars distributed throughout the 10 arcsecond field of view, and a local photometric correction process to account for photometric measurement biases introduced by PSF variability across the field of view. Our photometric methods achieve approximately 3% photometric precision across the observing time baseline for stars as faint as magnitude 15 in *K\'*. Our total sample of stars studied in this project was 563 stars.

In [Gautam+ (2019)][Gautam+2019], we performed a periodicity search on all 563 stars in our sample. This periodicity search identified the two known eclipsing binary systems at the GC: IRS 16SW and S4-258. The search also identified a new detection of periodic photometric variability in the star S2-36. S2-36 has a photometric variability period of 39.43 days.

<figure>
	<img src="./fit_amp_bs_sig.png" title="Periodic stars" />
    <figcaption>Significance of periodicity detection of GC stars in Gautam+ (2019) sample. The two known eclipsing binaries, IRS 16SW and S4-258, were detected significantly, as well as a newly detected periodic variable, S2-36. (Gautam+ 2019)</figcaption>
</figure>

[Gautam+2019]: https://ui.adsabs.harvard.edu/abs/2019ApJ...871..103G/abstract

## Periodic variability most consistent with a red giant, ellipsoidal binary at the Galactic center

To determine the astrophysical cause of variability, we used additional observations taken in *H*-band to obtain an estimate of the observed color of the periodic star.

<figure>
	<img src="./S2-36_color.png" title="S2-36 color" />
    <figcaption>Measurement of near-infrared color for the star S2-36, using observations in <em>K'</em> and in <em>H</em>. The blue line and bands represent the mean magnitude and uncertainty, respectively, at each wavelength band. (Gautam+ in prep.)</figcaption>
</figure>

Combining observations of the observed flux and color, we tested known sources of periodic variability at the observed photometric variability period: 2nd, 3rd, and 4th order pulsations in giants, Type I and Type II Cepheids, and red giant ellipsoidal binaries (aka sequence E variable). Each of these known sources of periodic variability have well-measured period-luminosity relationships (PLRs) in near-infrared, predicting intrinsic color and luminosity at the observed period. Comparing the observed color with the intrinsic color estimates a line-of-sight extinction towards the star, while the comparing the observed flux with the luminosity and extinction estimates a distance to the star. From these PLRs, the distance and extinction is most consistent with the GC under the red giant ellipsoidal binary (sequence E) hypothesis. The next closest hypothesis, Type II Cepheid, is inconsistent with GC distance by more than 3 sigma.

<figure>
	<img src="./ext_dist_spread.png" title="S2-36 extinction and distance spread" />
    <figcaption>Distance and extinction estimates from the observed S2-36 color under the red giant ellipsoidal binary hypothesis (red) and the Type II cepheid hypothesis (blue). The star's observed colors are most consistent with the Galactic center under the ellipsoidal binary hypothesis. (Gautam+ in prep.)</figcaption>
</figure>

## Binary light curve modeling

We have built new software ([PHOEBE Phitter](https://github.com/abhimat/phoebe_phitter)) to model the observed light curve of S2-36 in *K\'*- and *H*-band to estimate physical parameters of the stars making up the binary system. We use [PHOEBE2](http://phoebe-project.org) to generate model binary systems and light curves. We derive stellar parameters of the binary components using [PopStar](https://github.com/astropy/PopStar). We draw stellar parameters and synthetic photometry from isochrones generated at stellar ages spanning 1 Gyr to 13.5 Gyr, and with metallicities between [Fe/H] = –1.5 and [Fe/H] = 0.5.

Best-fit models 