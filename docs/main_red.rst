Main science / standard reduction
*********************************

The reduction of standard star and science data proceeds similarly, up to the
point where either the data are summed to a 1D spectrum, which is used for flux
calibration, or are converted to flux units via an existing calibration before
producing a datacube. In this example, we reduce the same standard star dataset
for both purposes.

* Scattered light subtraction proceeds as for the :ref:`flats`, but re-using
  the inter-block gaps that were identified at that stage:

  .. code-block:: none

    gfscatsub pxprgS20140919S0059 prgS20140919S0060_gaps
    gfscatsub pxprgS20140919S0062 prgS20140919S0061_gaps

  This could also be folded into the ``gfreduce`` call below (with
  ``fl_scatsub+``), except that I have added a step in between, to deal with a
  specific artifact in this dataset.

* Reset rows affected by "saturation banding" to zero. These were not dealt
  with in the bad pixel mask because there are no good pixels between which to
  interpolate and zeroing them at an earlier step would lead to negative values
  after subtracting scattered light.

  .. code-block:: none

    imreplace bpxprgS20140919S0059[sci,5][*,3031:3069] 0.
    imreplace bpxprgS20140919S0062[sci,5][*,3031:3069] 0.

* Perform most of the remaining basic "science" reduction, namely QE
  correction, mosaicking, extraction, flat fielding, wavelength rectification
  and sky subtraction. This does not need to be interactive, as we're mostly
  re-using existing calibrations.

  .. code-block:: none

    gfreduce bpxprgS20140919S0059 reference=prgS20140919S0060_init qe_refim=eprgS20140919S0080 response=eqbprgS20140919S0060_flat wavtraname=eprgS20140919S0080 fl_addmdf- fl_over- fl_trim- fl_bias- fl_qecorr+ fl_extract+ fl_gsappwave+ fl_wavtran+ fl_skysub+ fl_fluxcal- trace- recen- fl_vardq+ fl_inter-
    gfreduce bpxprgS20140919S0062 reference=prgS20140919S0061_init qe_refim=eprgS20140919S0081 response=eqbprgS20140919S0061_flat wavtraname=eprgS20140919S0081 fl_addmdf- fl_over- fl_trim- fl_bias- fl_qecorr+ fl_extract+ fl_gsappwave+ fl_wavtran+ fl_skysub+ fl_fluxcal- trace- recen- fl_vardq+ fl_inter-

  You might want to specify values for the ``w1`` & ``w2`` parameters, to
  restrict the output to some clean spectral range where both IFU slits have
  signal. Without this, the resulting datacubes have incomplete image planes at
  the ends of the spectra, covering only part(s) of the field. The ``dw``
  parameter can also be useful if you want to enforce a common sampling for all
  your data.

