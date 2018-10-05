Bad pixel mask
**************

* Create an empty bad pixel mask for detector defects such as bad columns:

  .. code-block:: none

    gemarith S20140919S0090_bias * 0 bpm_2x1 outtype="ushort"
    for i in range(1, 13):
      iraf.hedit("bpm_2x1[sci,%d]" % i, "EXTNAME", "DQ", update=yes, verify=no)

  (Press enter at the "..." prompt.)

  In a CL script, you would instead use CL syntax for the loop:

  .. code-block:: none

    for (i=1; i < 13; i+=1)
      hedit("bpm_2x1[sci,"//i//"]", "EXTNAME", "DQ", update+, verify-)

* Sometimes CL scripts are known to crash when header updates happen too fast
  for IRAF's FITS kernel. In a script, add this line after ``hedit``, to force
  it to catch up:

  .. code-block:: none

    flpr

* Look at the bias-subtracted data (especially the science exposures) and set
  any systematically-bad regions to 1 in the mask image:

  - Bad column on a saturated feature:

    .. code-block:: none

      imreplace bpm_2x1[dq,5][120:122,3027:4176] 1

  - Mask some artifically-bright rows & columns at the edges of each chip.
    The bright columns eventually get removed during mosaicking & extraction
    anyway, but dealing with them earlier can sometimes improve scattered light
    fitting and fibre tracing before we get to that point.

    .. code-block:: none

      for i in range(1, 13):
        iraf.imreplace("bpm_2x1[dq,%d][*,4172:4176]" % i, 1)

      imreplace bpm_2x1[dq,1][1:10,*] 1
      imreplace bpm_2x1[dq,4][247:256,*] 1
      imreplace bpm_2x1[dq,5][1:10,*] 1
      imreplace bpm_2x1[dq,8][247:256,*] 1
      imreplace bpm_2x1[dq,9][1:10,*] 1
      imreplace bpm_2x1[dq,12][247:256,*] 1

    You have to write ``iraf.`` before the first ``imreplace`` because the
    for loop puts the PyRAF interpreter in Python mode.

  Hamamatsu data from other eras (between multiple rounds of work on detector
  problems) have different quite artifacts. New data no longer have the above
  "saturation banding" problem (which got worse in 2015A) or the CCD2 noise
  problem from 2016B -- but the number of saturated columns seems to be greater
  now and bias roll-off along columns is quite strong.

* Attach the bad pixel mask to the bias-subtracted data and interpolate over
  detector defects (using a fairly low order continuum-type fit along each row):

    .. code-block:: none

      addbpm rg@all.lis bpm_2x1

      iraf.gemfix.grow=1.5
      iraf.gemfix.axis=1

      gemfix rg@all.lis prg@all.lis method="fit1d" bitmask=1 order=9 fl_inter-

