.. _flexure:

Wavelength flexure correction
*****************************

If (as in this example), you're using a baseline arc lamp observation, taken in
twilight or daytime, it's advisable to correct for any zero-point error(s) in
your wavelength solution(s) using sky lines. There is often at least one strong
line that can be used for this purpose, except at the bluest settings. [#f1]_

* One convenient way to measure a zero-point correction at usable S/N is to
  re-use the sky spectrum that ``gfskysub`` / ``gfreduce`` averages over the
  background IFU field and subtracts from your data, which is recorded in its
  own FITS extension of the output file. When using more than one sky line,
  wavelength offsets can be measured with respect to a line list using the IRAF
  task ``rvidlines``:

  .. code-block:: none

    iraf.rvidlines.observatory="gemini-south"
    iraf.rvidlines.coordlist="../scripts/skylines"
    iraf.rvidlines.nsum=1
    iraf.rvidlines.ftype="emission"
    iraf.rvidlines.fwidth=5.
    iraf.rvidlines.cradius=5.
    iraf.rvidlines.threshold=7.
    iraf.rvidlines.minsep=2.5
    iraf.rvidlines.logfile="F110_rvid.log"
    rvidlines steqbpxprgS20140919S0059.fits[sky]
    rvidlines steqbpxprgS20140919S0062.fits[sky]

  In the task's plot window, press ``l`` to try to match the line list
  automatically and then ``q`` to exit (or ``?`` for help).

  The output measurements are recorded in ``F110_rvid.log``, from which the
  -Zobs value can be multiplied by the approximate wavelength of 6300A to
  obtain corrections to the values of the WCS keyword CRVAL1. In this case, the
  sub-Angstrom corrections are only a few times the measurement errors, but
  nevertheless improve the accuracy. One could also measure just the 6300A line
  (eg. with the ``p p`` option in ``implot``).

  If you have a usable sky line but cannot identify its wavelength reliably,
  you can still correct any `differences` in its wavelength between exposures,
  permitting accurate registration when eventually mosaicking the datacubes.

* Add your measured zero-point corrections (with the opposite sign from Zobs)
  to the world co-ordinate system coefficients in the image headers:

  .. code-block:: none

    hedit steqbpxprgS20140919S0059.fits[sci] CRVAL1 5600.058 update+ verify-
    hedit steqbpxprgS20140919S0062.fits[sci] CRVAL1 5598.171 update+ verify-

* You can overplot spectra from both images using ``implot`` with its
  ``wcs=world`` option, to verify their alignment (eg. type ``l 300 340`` to
  plot an average over a sky block, ``:o`` to overlay the next plot,
  ``:i filename[sci]`` to load the next image and ``l 300 340`` again).

When in doubt, it's a good idea to take arc exposures on the sky with your
science data and flats, especially at blue wavelengths (despite adding several
minutes to each observing cycle), allowing you to skip this section.

.. [#f1] At red settings on faint targets, it's sometimes preferable to use sky
         lines directly for wavelength calibration, instead of the arc, but we
         won't go into that here.

