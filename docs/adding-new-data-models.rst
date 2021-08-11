.. _adding-new-data-models:

======================
Adding new data models
======================

**Obs-suite/level1a** has the capacity to map any type of data to the C3S Common Data Model format. Specifics about this data format can be found in the following `site <https://glamod.github.io/cdm-obs-documentation/index.html>`_.

The main task of this level of processing is to map data from ICOADS using schemas and common data model mappers already contained within the `mdf_reader <https://glamod.github.io/mdf_reader_documentation/>`_ and `cdm_mapper <https://glamod.github.io/cdm_mapper_documentation/>`_ tools. However, you can add new schemas and mappers **of your own making** to the tools, in order to process a specific ``sid-dck`` or an ICOADS-data collection.

The purpose of this page is to guide you step by step on how to add a new common data model for a specific collection, from the creation of a new schema (to read-in metadata with the ``.imma`` format), to run the :py:func:`level1a` processing level for that new CDM addition.

The first step probably is to select a source and deck that needs mapping; a list of pending data collections to map can be found in the following `repository <https://github.com/DavidBerryNOC/C3S_ICOADS_tracker/projects/1>`_. The diagram below shows an overview of the steps to follow:

.. figure:: _static/images/workflow_cdm_new.svg
    :width: 45%

    Simplified workflow of the steps needed to add a new CDM.

We recommend that you change the `mdf_reader <https://glamod.github.io/mdf_reader_documentation/>`_ and `cdm_mapper <https://glamod.github.io/cdm_mapper_documentation/>`_ repositories outside of the ``../glamod-marine/obs-suite/`` folder in JASMIN. You can either use another folder directory within JASMIN or just make a local copy of the two tools in your PC, this will facilitate to create, edit and test all the ``.json`` files that compose a schema and a cdm mapper.

To install the **mdf_reader** and **cdm_mapper** on your local PC (or in any other directory) you should follow the instructions in the following `documentation website <https://glamod.github.io/cdm_mapper_documentation/tool-set-up.html>`_. You will have to make *another python environment* just to run those tools separate from the **obs-suite** code.

Once you have set up the tools in another working directory, you are ready follow the instructions below. We will use **the US Marine Meteorological Journals Collection (MMJ 1878-1894)** ``125-704`` as an example for the following tutorial:

.. warning:: Once you have cloned and successfully installed the other two repositories you can **delete the files corresponding to deck 704 in the mdf_reader and cdm folders** so you can re-make them by following the steps below. Delete the folders ``../mdf_reader/data_models/lib/imma1_d704`` and ``../cdm/lib/mappings/icoads_r3000_d704``

.. toctree::
   :maxdepth: 2
   :caption: Steps:

   build-new-schema.rst
   build-new-cdm-mapper.rst
   last-steps.rst


