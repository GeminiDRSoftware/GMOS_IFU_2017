Cosmic ray rejection
********************

We continue reducing the science target / standard star data by detecting and
removing cosmic ray flux. The optional "LA Cosmic" is more effective for
fainter targets than the old ``gscrrej`` option. With bright IFU spectra in
particular, however, it can cause areas of spurious rejection on real features,
so you should check (and possibly tune) which pixels get flagged. When reducing
flux standards, you could set more conservative rejection parameters and do
just enough cleaning that the results will not be biased by cosmic ray flux.

This step should be performed early on, 1. because operations like resampling
or changes in S/N would undermine the detection, 2. to avoid other steps being
affected by or spreading cosmic ray flux and 3. to avoid repetition when tuning
other steps, since I think it's the slowest one in the whole process.

* Run LA Cosmic [#f1]_ on the target spectra via the gemtools wrapper script
  (you must first have installed ``lacos_spec.cl``):

  .. code-block:: none

    gemcrspec prg@std.lis xprg@std.lis fl_vardq+ verb+

  This can also be called via the ``fl_crspec`` option in ``gfreduce``.

* Compare each resulting DQ plane with the corresponding SCI input, especially
  where the signal is strong, to verify that cosmic rays have been detected
  accurately without systematically rejecting other regions. A few isolated
  bright or dark pixels will get flagged in addition to cosmic rays, but only
  concentrated areas/rows of rejection are a cause for concern (if they contain
  signal of interest). For example:

  .. code-block:: none

    gdisplay prgS20140919S0059 1
    gdisplay xprgS20140919S0059 2 sci_ext="DQ"

  You can also ``display`` each extension in turn, instead of using
  ``gdisplay``.

* Re-clean the pixels flagged by LA Cosmic, since ``fixpix`` usually does a
  slightly better job than the detection algorithm's own median filter:

  .. code-block:: none

    gemfix xprg@std.lis pxprg@std.lis method="fixpix" bitmask=8

The alternative cleaning method of stacking >=3 identical exposures with pixel
rejection is often inappropriate for IFU science observations -- which tend to
use long exposures to obtain high enough S/N -- because changes in airmass and
flexure cause the distribution of signal over the IFU field and detector to
vary between exposures.


.. [#f1] van Dokkum (2001), PASP 113, 1420,
         http://www.astro.yale.edu/dokkum/lacosmic.

