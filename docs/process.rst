Data reduction outline
**********************

The nominal reduction process used here for science target exposures can be
summarized as follows (not showing calibration processing). This includes more
steps than the rather old "``gmosexamples IFU_2slit``".

  .. figure:: process_chart.svg
     :scale: 80%
     :align: center

     Outline of the main data reduction process for science target exposures
     (which may be modified or split up a bit to deal with specific problems).

In the actual example to be presented, a few steps have been inserted or split,
for specific reasons, and the calibration processing is mixed in.

After each step of this process, the filename gets prefixed with an additional
letter (eg. "x" for cosmic ray rejection or "e" for extraction), making it
relatively easy to understand the processing state of intermediate files.

During the reduction, the data evolve between three main formats:

  1. The raw image format, containing 12 FITS extensions (3 CCDs x 4
     amplifiers):

    .. figure:: std_det_mos_zoom.png
       :scale: 50%
       :align: center

       A partly-reduced IFU spectrum of a star in raw detector
       co-ordinates, after mosaicking the 12 amplifiers.

  2. Images containing one extracted 1D fibre spectrum per row:

    .. figure:: std_rss.png
       :scale: 50%
       :align: center

       Row-stacked 1D fibre spectra, near the end of the reduction process.
       The spectra are first extracted into one FITS extension per slit, then
       the 2 slits are stacked vertically like this after wavelength
       rectification.

  3. 3D data "cubes":

    .. figure:: cube.png
       :scale: 30%
       :align: center

       Non-artist's misimpression of a reduced x-y-λ datacube, after resampling
       the extracted 1D spectra onto a 3D grid.

