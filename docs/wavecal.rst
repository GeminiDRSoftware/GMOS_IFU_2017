.. _wavecal:

Wavelength calibration
**********************

* Extract the arc lamp spectra non-interactively, re-using the fibre traces
  from the flat as a reference. Data quality propagation is enabled here just
  because it results in better chip gap interpolation.

  .. code-block:: none

    gfreduce prgS20140919S0080 reference=prgS20140919S0060_init fl_addmdf- fl_over- fl_trim- fl_bias- fl_extract+ fl_gsappwave+ fl_wavtran- fl_skysub- fl_fluxcal- trace- recen- fl_vardq+ fl_inter-
    gfreduce prgS20140919S0081 reference=prgS20140919S0061_init fl_addmdf- fl_over- fl_trim- fl_bias- fl_extract+ fl_gsappwave+ fl_wavtran- fl_skysub- fl_fluxcal- trace- recen- fl_vardq+ fl_inter-

  - You may first want to compare the bias-subtracted arc and flat with
    ``gdisplay`` and check that the latter is actually an accurate tracing
    reference, with no major difference in flexure. Misalignment is likely to
    affect the wavelength accuracy only slightly, due to minimal fibre-to-fibre
    differences.

* Make a copy of the GMOS CuAr line list that we can edit as needed for this
  dataset:

  .. code-block:: none

    copy gmos$data/CuAr_GMOS.dat scripts$CuAr_F110.dat

  It can be helpful to remove lines outside the wavelength range of your
  spectra (in this case 560-700nm) from the list at this stage.

* Set some parameter defaults to be more appropriate for calibrating 2x1-binned
  IFU spectra (rather slit spectra):

  .. code-block:: none

    iraf.gswavelength.coordlist="scripts$CuAr_F110.dat"
    iraf.gswavelength.nsum=1
    iraf.gswavelength.gsigma=0.
    iraf.gswavelength.cradius=5.
    iraf.gswavelength.minsep=1.5
    iraf.gswavelength.step=1
    iraf.gswavelength.low_reject=2.5
    iraf.gswavelength.high_reject=2.5

* Calibrate the arcs in wavelength. The results must be checked interactively,
  but you can again ignore the second exposure until you finish iterating on
  the first one.

  .. code-block:: none

    gswavelength eprgS20140919S0080 order=5 fl_inter+
    gswavelength eprgS20140919S0081 order=5 fl_inter+

  - The first plot to appear shows the spectrum of the middle fibre with
    automatic line identifications overlaid (assuming they were found
    automatically).

    .. figure:: wavecal_lines.png
       :scale: 50%
       :align: center

       Final line identifications for the middle fibre of slit 1.

    You can use the ``+`` and ``-`` keys to cycle through line identifications
    and see their matching list wavelengths (3rd number).

    It's best to delete the faintest lines (eg. < 100-200 counts), as long as
    you have at least (say) 10-20 remaining over the spectral range. You can
    zoom with ``w e e`` (as for extraction) and comment out any unwanted
    entries in the line list.

    If no line identifications are shown, you will have to mark 2 lines
    manually and enter their wavelengths, then press ``l`` to match the rest of
    the line list (press ``?`` for further information).

  - Pressing ``f`` shows the fitted wavelength solution (see ``?`` for more
    options). Any line measurements excluded by the rejection algorithm are
    shown with a box around them; you may want to return to the first plot by
    pressing ``q`` and have a closer look at these, commenting them out from
    the line list if they look blended or border a chip gap etc.

    .. figure:: wavecal_fit.png
       :scale: 50%
       :align: center

       Wavelength solution for the middle fibre of slit 1.

    Here, I have commented out lines at 5606, 5772, 5802, 6155, 6172, 6364,
    6369, 6466 & 6483A. The RMS of 0.07A seems perfectly reasonable for 2x1
    binning (at about 1/14th of a pixel). It will vary a bit for the other
    fibres.

  - Press ``q`` (twice if in fitting mode) to move onto the next fibre
    spectrum. You can then generally type ``NO``, to skip all the other fibres
    -- just make sure the first one plotted looks good and that the status
    lines printed for the others show most of the same arc lines being found
    and generally reasonable RMS values.

* Resample the calibrated arcs onto linear wavelength co-ordinates, to check
  the solution (ie. that the lines are straight):

  .. code-block:: none

    gftransform eprgS20140919S0080 wavtraname=eprgS20140919S0080
    gftransform eprgS20140919S0081 wavtraname=eprgS20140919S0081

    displ teprgS20140919S0080[sci] 1  # (for example)

  There is often some sub-pixel fuzziness, due to small differences in the
  solutions for individual fibres (especially at the very ends, where the
  fits are underconstrained), but it's not practical to tweak every fibre
  individually. There are also some darker rows, since the data have not been
  flat fielded, as well as cosmetics etc. You may see a few second order or
  ghost lines superimposed, which are curved, diffuse or appear only in one
  slit; these can be ignored here, as long as they did not cause
  misidentifications in ``gswavelength``.

  This particular spectrum exhibits faint interpolation "ringing" (dark
  columns) around the brightest lines, due to the 2x1 binning.

