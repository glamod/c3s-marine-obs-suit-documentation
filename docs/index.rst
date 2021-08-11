.. C3s summary documentation master file, created by
   sphinx-quickstart on Wed Jul 14 10:17:42 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

C3S marine-obs-suite documentation
==================================

The Marine Observations Suite (or just obs-suite) is part of the code implemented to produce the data deliveries for the C3S Marine In Situ Component.

Instructions to run the full set of suites, including this one, are available in this `technical guide <https://github.com/glamod/glamod-marine-processing/blob/master/obs-suite/docs/build/latex/marineobservationssuite.pdf>`_  and the scientific background of the processing can be found in the latest version of the `Marine data user guide <https://github.com/glamod/marine_user_guide/blob/master/C3S_Marine_PUG/C3S_D311a_Lot2.3.4.4-2021_Final_Version_Marine_User_Guide.pdf>`_. However, the purpose of this documentation is to facilitate the understanding of the technical manual by covering the following topics:

- The update of old code paths and repositories found in the marine observation suit manual.
- Explain how to deploy and run the obs-suite code in the `JASMIN cluster services <https://help.jasmin.ac.uk/category/158-getting-started>`_
- Explain how to create and integrate a new `mdf_reader schema <https://glamod.github.io/mdf_reader_documentation/>`_ and `common data model mapper <https://glamod.github.io/cdm_mapper_documentation/>`_ to the **obs-suite** code for a specific data collection from `ICOADS <https://icoads.noaa.gov/>`_ (e.g. a new **SID-DCK**).
- Explain how to run the first level of processing: :py:func:`level1a` after adding the new schema and cdm mapper.
- How to use the ``mdf_reader`` tool and ``cdm_mapper`` tool to obtain missing ``code_tables`` in the metadata.
- How to solve conflicts after adding schemas and mappers and dealing with **invalid/excluded** data.


.. toctree::
   :maxdepth: 2
   :caption: Contents:

   obs-suite-set-up.rst
   release-workflow.rst
   adding-new-data-models.rst
   issues-with-data-models.rst


About
-----

:Version:

:Citation:

:License:

:Authors:
   David Berry, Irene Perez Gonzalez and Beatriz Recinos


.. image:: _static/images/logos_c3s/logo_c3s-392x154.png
    :width: 25%
    :target: https://climate.copernicus.eu/
.. image:: _static/images/logos_c3s/LOGO_2020_-_NOC_1_COLOUR.png
    :width: 25%
    :target: https://noc.ac.uk/
.. image:: _static/images/logos_c3s/icoadsLogo.png
    :width: 20%
    :target: https://icoads.noaa.gov/