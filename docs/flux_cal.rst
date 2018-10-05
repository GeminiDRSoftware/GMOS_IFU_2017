Spectrophotometric flux calibration
***********************************

* Integrate over the reduced standard star spectra spatially, to extract a 1D
  spectrum from each exposure. This simple summation is not optimal in terms of
  S/N, or for avoiding residual artifacts, but it's more robust over a large
  dynamic range than a weighted sum with pixel rejection and is good enough for
  a bright target.

  .. code-block:: none

    gfapsum steqbpxprgS20140919S0059 combine="sum" reject="none" scale="none" zero="none" weight="none" lthreshold=-25. fl_inter-
    gfapsum steqbpxprgS20140919S0062 combine="sum" reject="none" scale="none" zero="none" weight="none" lthreshold=-25. fl_inter-

  The threshold here excludes any abnormally-negative values, just because
  that's easy to do (without cutting close enough to zero affect the statistics
  much).

* Combine the 2 wavelength settings into a single spectrum. One could instead
  calibrate each setting separately, but it won't make much difference for a
  small dither and it's convenient to work with a single calibration.

  .. code-block:: none

    gemscombine asteqbpxprgS20140919S0059,asteqbpxprgS20140919S0062 asteqbpxprgS20140919S0059_add

  Note that the three spectra have almost the same wavelength ranges in this
  case, even though a 5nm dither was used; this is because the full waveband of
  the r filter (which is what gets extracted from the raw images) fits on the
  detector mosaic.

* Find the appropriate flux standard table and make a copy that can be edited
  as necessary. Most of the standards used for GMOS-S are from Hamuy et al.
  (1994) and have tabulated fluxes in IRAF's ``onedstds$ctionewcal`` directory
  (type ``page onedstds$README`` for more information). This star is Feige 110,
  as you can see with ``imhead asteqbpxprgS20140919S0059.fits[0]``.

  .. code-block:: none

    copy onedstds$ctionewcal/f110.dat scripts$

  Edit this table and comment out the 6850 & 6900A bands, which Hamuy et al.
  flag as being affected by telluric absorption (which is not constant).

* Calibrate the instrumental sensitivity curve interactively, first determining
  which spectral bins to compare with the flux table and then fitting a
  low-order function to the resulting ratios, expressed in magnitudes.

  .. code-block:: none

    iraf.gsstandard.observatory="gemini-south"
    iraf.gsstandard.extinction="onedstds$ctioextinct.dat"
    gsstandard asteqbpxprgS20140919S0059_add f110_std f110_sens starname=F110 caldir=scripts$ order=4 fl_inter

  The optional atmospheric extinction correction (for the nearby Cerro Tololo)
  amounts to 0.4 mag/airmass in absolute terms at the blue end of the g filter,
  decreasing with wavelength, but can make minimal difference to the end result
  where the science data and standard are taken at similar airmasses. Note
  that baseline standards are often taken through cloud (which is assumed to be
  grey), in which case only the shape of the correction is significant.

  The first (``standard``) plot window looks something like this:

  .. figure:: standard.png
     :scale: 50%
     :align: center

     Determination of flux bins to use for spectrophotometric calibration
     in the IRAF task ``standard`` (via ``gsstandard``).

  The "wiggles" here are probably due to some combination of the GMOS optics
  (especially the `r filter
  <http://www.gemini.edu/sciops/instruments/gmos/filters/r_G0326_plot_Jun2017.pdf>`_)
  and fringing. Unfortunately, this structure seems difficult to remove from
  these observations at either the flat-fielding or flux calibration steps,
  without making other compromises; it's not a large effect quantitatively, but
  distorts the continuum shape noticeably. Someone with more expertise in flux
  calibration may be able to address this better than here (it's only
  IFU-related insofar as the IFU-2 uses broad-band filters).

  Once happy with the bin selection, you can press ``q`` to continue.

  The next (``sensfunc``) plot looks like this:

  .. figure:: sensfunc.png
     :scale: 50%
     :align: center

     Fitting an instrumental sensitivity function in ``sensfunc`` (via
     ``gsstandard``).

  Press ``?`` to see fitting options. Again, the values exhibit some modulation
  with respect to the low-order fit, which is not removed with a slightly
  higher order. Press ``q`` to save the sensitivity function.

* Apply the newly-derived flux calibration back to the standard spectrum
  (just to check the results) and to the "science" data, in order to flatten
  the spectral response and convert from electrons to units of ergs/s/cm^2/A.
  Here, we simply calibrate the standard with itself, since we're using it as
  an example.

  .. code-block:: none

    iraf.gscalibrate.observatory="gemini-south"
    iraf.gscalibrate.extinction="onedstds$ctioextinct.dat"
    gscalibrate asteqbpxprgS20140919S0059_add sfunc=f110_sens fl_vardq+ fl_ext+
    gscalibrate steqbpxprg@std.lis sfunc=f110_sens fl_vardq+ fl_ext+

  Note that ``gscalibrate`` applies an arbitrary scaling of 10^15 by default
  (to avoid later numerical errors), which you will eventually need to divide
  by to get true flux units.

* If your standard was taken in photometric conditions but moderate-to-poor
  seeing and the absolute calibration accuracy is important, you may want to
  estimate any flux loss outside the IFU field by reconstructing a data cube
  of the star observation, summing over it in wavelength, fitting a PSF,
  determining the sigma cut-offs at the edges and, for example, looking up
  the integral of a 2D Gaussian between those limits (which is probably an
  underestimate but gives you some idea); then you can multiply your fluxes
  by the proportion of standard flux in the field, as a correction.

