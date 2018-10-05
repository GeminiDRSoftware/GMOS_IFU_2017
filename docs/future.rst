Future improvements
*******************

* I have developed a personal Python package that can be used to script GMOS
  IFU data reduction more efficiently than in CL:

  - Developer-orientated presentation: https://zenodo.org/record/50991
  - API documentation: http://ndmapper.readthedocs.io/en/latest
  - Software repository: https://github.com/jehturner/ndmapper

  (This pre-dates the recent AstroData re-write, which now looks quite similar
  to my DataFile class.)

  Most of the reduction steps are still Gemini IRAF wrappers at present, with a
  small number re-implemented in Python (mainly for cleaning cosmetics).

  This is what I'm now using for my own science projects ... but 1. it's not
  our official code, 2. it's not trivial to set up everything needed (something
  I will address) and 3. some aspects (such as how to override calibration
  matches) are currently still a bit fiddly and would be difficult to explain
  until streamlined and documented better.

* The official Gemini Python pipeline should eventually support reduction of
  spectroscopic data (perhaps using a combination of Gemini IRAF wrappers and
  new Python code at first). I also hope to do more work on this soon...

  - Extraction of spectra and wavelength calibration are likely priorities for
    working with GMOS IFU data.

