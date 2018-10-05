.. _flats:

Flat field calibration
**********************

We have already extracted some flat field spectra as a reference for tracing
fibres, but those data are still contaminated by scattered light and have not
been corrected for differences in quantum efficiency variation between chips,
so we now correct for those effects and re-extract the resulting spectra before
creating normalized flats.

* Identify unilluminated regions in the gaps between blocks of fibres, using
  header information from the previous extraction:

  .. code-block:: none

    gffindblocks prgS20140919S0060 prgS20140919S0060_init prgS20140919S0060_gaps
    gffindblocks prgS20140919S0061 prgS20140919S0061_init prgS20140919S0061_gaps

  The output text files can be edited manually in the event of problems with
  scattered light subtraction; the regions don't need to cover every single
  unilluminated row, but they must not contain significant signal from the
  fibres and should span as much of the detector as possible. For GMOS-S, any
  16th region at the top may need removing, as there is not enough space to
  measure the background at the top of the detector. Intervention is often
  unnecessary.

* For each amplifier, fit a low-order surface to the scattered light level in
  the background and subtract the result:

  .. code-block:: none

    iraf.gfscatsub.xorder="2,2,2,2,2,2,3,2,2,2,2,2"
    iraf.gfscatsub.yorder="3"

    gfscatsub prgS20140919S0060 prgS20140919S0060_gaps
    gfscatsub prgS20140919S0061 prgS20140919S0061_gaps

  It would likely be better to fit a surface to the full image mosaic,
  enforcing continuity, but that's not how the scripts currently work... You
  may have to experiment a bit with the orders to get a good model background
  approximation, subtracting the input image from the output with ``gemarith``
  to see what the fit looks like. The correction will never be perfect, but the
  gap levels should be significantly closer to zero after this step than
  before, even if there are some residuals (eg. ~200 counts in a flat instead
  of ~3000 originally).

* Correct for the different variations in quantum efficiency between CCDs,
  using a look-up table and our existing wavelength calibration:

  .. code-block:: none

    gqecorr bprg@flat.lis refimages=eprg@arc.lis fl_vardq+

  This should reduce any significant discontinuities between adjacent chips,
  though, again, the correction is not always perfect (nor is interpolation of
  the gap itself).

* Repeat flat-field extraction using the fully-reduced data. We could re-trace
  the fibres here, but the traces should be minimally affected by the removal
  of background on larger scales and the intention is to extract all the
  exposures (including the arcs we've already done) using identical traces:

  .. code-block:: none

    gfreduce qbprgS20140919S0060 reference=prgS20140919S0060_init fl_addmdf- fl_over- fl_trim- fl_bias- fl_extract+ fl_gsappwave+ fl_wavtran- fl_skysub- fl_fluxcal- trace- recen- fl_vardq+ fl_inter-
    gfreduce qbprgS20140919S0061 reference=prgS20140919S0061_init fl_addmdf- fl_over- fl_trim- fl_bias- fl_extract+ fl_gsappwave+ fl_wavtran- fl_skysub- fl_fluxcal- trace- recen- fl_vardq+ fl_inter-

* Normalize the flats, by fitting and dividing out the continuum:

  .. code-block:: none

    gfresponse eqbprgS20140919S0060 eqbprgS20140919S0060_flat skyimage="" order=45 fl_fit- sample="*" fl_inter- # wavtraname=eprgS20140919S0080
    gfresponse eqbprgS20140919S0061 eqbprgS20140919S0061_flat skyimage="" order=45 fl_fit- sample="*" fl_inter- # wavtraname=eprgS20140919S0081

  This step can also be done interactively, to optimize the continuum fit. The
  relatively-high order used here removes most large-to-medium-scale structure,
  some of which may be optical structure or fringing that could legitimately be
  preserved to help flatten ths final science spectrum but is difficult to
  disentangle from the lamp spectrum; you may be able to improve on this result.

  I am not using twilight flats because I see no evidence that they help
  significantly with flatness over the small GMOS field, whereas they do
  introduce flexure-related errors in the fibre-to-fibre response.

  My `ifudrgmos` version of the GMOS package, posted on the Data Reduction
  forum, has a more accurate normalization algorithm:
  http://drforum.gemini.edu/topic/gmos-ifu-data-reduction-scripts. This has an
  additional parameter for a wavelength-calibrated reference, without which
  some datasets may exhibit spatial curvature and/or flux discontinuities
  between the two halves of the 2-slit IFU field. This has not replaced the
  standard version to date because it has the option to use twilight flats
  disabled (and `ifudrgmos` is still based on 1.13, so won't support GMOS-N
  Hamamatsu data yet).

