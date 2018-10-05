Getting started & organizing files
**********************************

Starting a PyRAF session
========================

* In ``red/``, start a PyRAF session and an image display tool, eg.:

  .. code-block:: sh

    > ds9 &
    > pyraf

* A provided ``loginuser.cl`` file automatically defines ``raw$`` &
  ``scripts$`` path variables, picks up the local LA Cosmic and configures
  image display for GMOS.

.. This also uses a simple gmosaic patch to deal with an oversight in the chip
   gap interpolation.

* Subsequent commands may be pasted into your PyRAF session and/or a CL
  script in a text editor (a complete script is provided for reference in
  ``scripts/ref/f110.cl``).
  When constructing a CL script or working in CL (rather than PyRAF), you must
  remove the ``iraf.`` prefix from any parameter settings (eg. below).

* Load all the packages needed by the reduction process:

  .. code-block:: none

    gemini
    gmos
    rv
    pyfu
    task gmosaic=home$../scripts/gmosaic.cl   # gap interpolation patched
    unlearn gemtools gmos                     # use reproducible parameter defaults

  - You can add these commands to ``loginuser.cl`` if you want (along with
    ``gdisplay.fl_paste`` below), so you don't have to repeat them when
    restarting your session. If pyraf fails to start as a result, you may need
    to comment out the section in your ``login.cl`` beginning ``if
    (deftask("samp") == yes) {``.

* Specify a log file for this target (optional) and display GMOS images using
  the more practical ``fl_paste+`` option:

  .. code-block:: none

    iraf.gmos.logfile="F110.log"
    iraf.gemtools.logfile="F110.log"

    iraf.gmos.gdisplay.fl_paste=yes

  Again, remove the ``iraf.`` here when pasting into a script.
  
* I believe the previous GMOS tutorial by Ricardo will discuss what GMOS data
  look like in general (otherwise I can elaborate a bit on that before moving
  on).


Organizing the data
===================

* Organize the data files into lists. Some (not all) reduction steps can use
  these to construct their input or output filenames, avoiding more repetition
  than necessary in our script.

  .. code-block:: none

    gemlist S20140919S 90-94 > slow_bias.lis
    gemlist S20140926S 178-182 > fast_bias.lis
    gemlist S20140919S 60-61 > flat.lis
    gemlist S20140919S 80-81 > arc.lis
    gemlist S20140919S 59,62 > std.lis
    concatenate flat.lis,arc.lis,std.lis all.lis

