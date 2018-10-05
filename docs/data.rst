Tutorial setup, data & directories
**********************************

* Make sure you have Anaconda installed, with the necessary Astroconda
  packages, as described at http://www.gemini.edu/node/12665.

* Make sure your shell session is configured to use the conda environment into
  which you installed the Astroconda packages (eg. "gemini" or "astroconda"):

  .. code-block:: sh

     > source activate <env>

* Unpack the data tarball somewhere under your home directory (producing a
  subdirectory of the same name).

  .. code-block:: sh

     > tar zxf GMOS_IFU_2017.tar.gz
     > cd GMOS_IFU_tut_2017/

* Copy LA Cosmic (from
  ``http://www.astro.yale.edu/dokkum/lacosmic/lacos_spec.cl``) into the
  ``scripts/`` subdirectory.

* Create an IRAF login.cl in the ``red/`` subdirectory:

  .. code-block:: sh

     > cd red/
     > mkiraf

  (Answer ``xterm`` or ``xgterm`` for the terminal type, not
  ``xterm-256color``.)

* The directories are structured as follows:

  =========    =====================================================
   raw/          All of the raw FITS files needed for the tutorial
   red/          Working directory for reducing data
   scripts/      Reduction scripts and look-up tables
  =========    =====================================================

