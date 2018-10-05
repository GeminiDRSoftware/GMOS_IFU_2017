Bias frames & subtraction
*************************

* Set some common parameter defaults for creating Hamamatsu bias frames and
  tell the script where to find raw data:

  .. code-block:: none

    iraf.gbias.rawpath="raw$"
    iraf.gbias.fl_over=yes
    iraf.gbias.fl_trim=yes
    iraf.gbias.biasrows="default"
    iraf.gbias.order=11
    iraf.gbias.low_reject=3.0
    iraf.gbias.high_reject=3.0
    iraf.gbias.niterate=5
    iraf.gbias.fl_vardq=yes

  The overscan fitting order is quite high for these Hamamatsu data, due to
  variable roll-off along columns (whose origin I don't understand well).

  For the EEV detectors, I used to fit a constant value to a small overscan
  section that was unaffected by their strong charge transfer contamination,
  but this now seems much less of an issue and I am fitting the full height of
  the overscan region.

* Generate stacked bias frames:

  .. code-block:: none

    gbias @slow_bias.lis S20140919S0090_bias fl_inter-
    gbias @fast_bias.lis S20140926S0178_bias fl_inter-

  It's best to run these commands interactively first time and inspect the
  overscan fit -- but this means pressing 'q' about 60 times each and it's not
  the most critical thing to show here, so I've left it out of the tutorial.

  If you do need to change any parameters after inspecting the fit, do it in
  the script above (to make the changes permanent), then you can disable
  ``fl_inter+`` when re-running the script later and get the same results.

* Subtract the bias from the raw data files and attach an MDF extension to
  each one. This can be done using only the the bias & MDF options in gfreduce.
  Subtracting the bias separately from other gfreduce steps allows inspection
  and manual creation of a BPM before proceeding further.

  .. code-block:: none

    iraf.gfreduce.rawpath="raw$"
    iraf.gfreduce.mdfdir="scripts$"
    iraf.gfreduce.fl_gscrrej=no
    iraf.gfreduce.biasrows="default"
    iraf.gfreduce.order=11
    iraf.gfreduce.low_reject=3.0
    iraf.gfreduce.high_reject=3.0
    iraf.gfreduce.niterate=5
    iraf.gfreduce.fl_fulldq=yes
    iraf.gfreduce.fl_vardq=yes

    gfreduce @flat.lis bias=S20140919S0090_bias fl_addmdf+ fl_over+ fl_trim+ fl_bias+ fl_extract- fl_gsappwave- fl_wavtran- fl_skysub- fl_fluxcal- fl_inter-
    gfreduce @arc.lis bias=S20140926S0178_bias fl_addmdf+ fl_over+ fl_trim+ fl_bias+ fl_extract- fl_gsappwave- fl_wavtran- fl_skysub- fl_fluxcal- fl_inter-
    gfreduce @std.lis bias=S20140919S0090_bias fl_addmdf+ fl_over+ fl_trim+ fl_bias+ fl_extract- fl_gsappwave- fl_wavtran- fl_skysub- fl_fluxcal- fl_inter-

  (You can either type each gfreduce command as a single long line or split
  them with the continuation character ``\`` for readability. Here I have kept
  them on one line just to make cutting & pasting easier for the tutorial).

