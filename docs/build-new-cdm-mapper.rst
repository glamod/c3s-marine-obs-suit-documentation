.. _build-new-cdm-mapper:

=============================
2. Build and add a cdm-mapper
=============================

Detail instructions on how to build and add a new cdm-mapper for any type of data can be found in the `cdm_mapper docs <https://glamod.github.io/cdm_mapper_documentation/how-to-register-a-new-data-model-mapping.html>`_. However, the process to create a **mapper** for a data collection from ICOADS can be simplified by copying an **already built in** cdm_mapper i.e., ``icoads_r3000``.

2.1 Set up your script
----------------------

After activating the correct python environment, you can start a new python console and create the following script by typing the commands below. Don't forget to change your working directory path first::

    import os
    import sys
    working_dir = '/to_your_designated_folder_where_you_installed_mdf_reader_and_cdm/'
    sys.path.append(working_dir)
    import mdf_reader
    import json
    import pandas as pd
    import cdm

To get familiar with the cdm tool you can read a raw `.imma` file from dck 704 as an example, for a subset of the data collected in 1878/10 and apply the existing ``icoads_r3000`` data model::


    data_file_path = os.path.join(working_dir, 'mdf_reader/tests/data/125-704_1878-10_subset.imma')
    schema_new = 'imma1_d704'
    data_new = mdf_reader.read(data_file_path, data_model = schema_new)

    attributes = data_new.atts.copy()
    name_of_model = 'icoads_r3000'

    cdm_dict = cdm.map_model(name_of_model, data_new.data, attributes, cdm_subset = None, log_level = 'INFO')

You can explore each table inside the ``cdm_dict``::

    cdm_dict['header']['data'].head()
    cdm_dict['observations-at']['data'].head()

2.2 Copy files from a pre-existent data model mapper
----------------------------------------------------

To construct the cdm-mapper for deck 704 you can copy a the default ``icoads_3000`` mapper and modify the necessary tables, python script and ``code_tables``. Open a shell and do the following::

    $ cd ../cdm/lib/mappings
    $ cp -r icoads_r3000 icoads_r3000_d704
    $ cd icoads_r3000_d704
    $ ls
        __init__.py            icoads_r3000.py        observations-sst.json
        __pycache__/           observations-at.json   observations-wbt.json
        code_tables/           observations-dpt.json  observations-wd.json
        header.json            observations-slp.json  observations-ws.json

.. warning::
    It is important that you change the ``icoads_r3000.py`` file name to the corresponding deck number (in this case ``icoads_r3000_d704.py``). The main folder of the data model and the ``.py`` script need to have the same name, so the cdm tool is able to recognize it.

Make sure you do::

    $ mv icoads_r3000.py icoads_r3000_d704.py

2.3 Modify the header table
---------------------------

For this deck we wont be changing much, we have very few data in the ``c99`` supplement that is useful. For future decks this might be very different and you might be able to extract more valuable information from the ``c99`` string. The following `documentation website <https://glamod.github.io/cdm_mapper_documentation/cdm-tables-mapping-files-and-descriptors.html#>`_ gives you a detail overview of all the descriptors that you might needed to successfully declare an element in a header/observation table.

- In the file ``header.json`` (after line 18) add the station name that it will be read from the ``c99_journal`` section of the supplemental data::

    "station_name": {
            "sections": "c99_journal",
            "elements": "ship_name"
        },

- For this deck we will also change the fields ``platform_sub_type`` and ``primary_station_id`` to add extra information from the supplemental data::

    "platform_sub_type": {
        "sections": "c99_journal",
        "elements": "rig",
        "code_table": "platform_sub_type"
    },
    "primary_station_id": {
        "sections": "c99_journal",
        "elements": "ship_name"
    },

- After modifying the header make sure you still have a valid json file, check this via an `online tool <https://jsonlint.com/>`_ or just make sure you use a json editor.

2.4 Modify the observations tables
----------------------------------

For this deck we will only add extra information to the sea-level-pressure observation table. You will need to modify only the file with the name ``observations-slp.json``.

- In the editor of your choice add the following lines after the ``crs`` (~ line 41), so your file looks like::

    "crs": {
        "default": 0
    },
    "z_coordinate": {
        "sections": "c99_journal",
        "elements": "baro_height",
        "transform": "feet_to_m",
        "decimal_places": 2
    },

- And the following after ``z_coordinate_type``, so it looks like::

    "z_coordinate_type": {
            "default": 0
        },
    "observation_height_above_station_surface": {
        "sections": "c99_journal",
        "elements": "baro_height",
        "transform": "feet_to_m",
        "decimal_places": 2
    },

Here we are reading information about the barometer height. The argument ``"transform": "feet_to_m"`` means that the code will look for this ``feet_to_m`` function inside the ``icoads_r3000_d704.py`` (imodel.py) script that we will modify in step 2.6.

- We will also read in the original units for these measurements **by replacing**::

    "original_units": {
        "default": 530
    },

- **With the following**::

    "original_units": {
        "sections": "c99_journal",
        "elements": "baro_units",
        "code_table": "baro_units",
        "fill_value": 9999
    },

To implement this modification we will need to add a ``code_table`` named ``baro_units``, in order to transform the original units code, to the C3S CDM corresponding key-code for such units. We will do this in the following step.

2.5 Add ``code_tables`` needed
------------------------------

In our ``header.json`` and ``observations-slp.json`` we are reading information about the type of rig (``platform_sub_type``) and barometer units (``baro_units``). For such information we need to add a ``code_table`` that can translate the original key code from ``c99``, to the C3S CDM code for that specific element. We need to add two files under ``../cdm/lib/mappings/icoads_r3000_d704/code_tables``:

- ``$ vim platform_sub_type.json``::

    {
     "01":26,
     "02":105,
     "03":106,
     "04":107,
     "05":108,
     "06":109,
     "99":26
    }

- ``$ vim baro_units.json``::

    {
      "1":1001,
      "2":1002,
      "3":1003,
      "4":9999,
      "5":1004
    }

When we declare these elements in the header/observation table, we are passing to the mapper function the name of these files, by using the descriptor ``code_table``. For an overview of all elements descriptors, please refer to the `cdm-mapper documentation <https://glamod.github.io/cdm_mapper_documentation/cdm-tables-mapping-files-and-descriptors.html#>`_

2.6 Add the functions needed to the ``icoads_r3000_d704.py``
------------------------------------------------------------

In our ``observations-slp.json`` we are also reading information about the height of the barometer. These measurements were originally made in feet but the C3S CDM format require that they are in meters. So we have to modify each measurement value by creating a conversion function ``feet_to_m``, that we will store in the ``icoads_r3000_d704.py`` python script. This script will host all conversion functions or any other function needed to modify the original data.

.. note:: In our `cdm-mapper documentation <https://glamod.github.io/cdm_mapper_documentation/cdm-tables-mapping-files-and-descriptors.html#>`_ this script is called the ``imodel.py`` script. For more information on the ``imodel.py`` check out `this section <https://glamod.github.io/cdm_mapper_documentation/cdm-tables-mapping-files-and-descriptors.html#defining-mapping-functions>`_ of the cdm_mapper docs.

Lets now add the function ``feet_to_m`` to the ``icoads_r3000_d704.py`` python script after the line 161::

        def feet_to_m(self, ds, float_type='float32'):
        ds.astype(float_type)
        return np.round(ds/3.2808, 2)

It is important that all functions are place under the ``class mapping_functions():`` line so they can be recognize by the mapping tool box.

2.7 Test your new CDM mapper
----------------------------

To test that you have added the files corrected. Re-start your python environment and python console and run the following lines::

    data_file_path = os.path.join(working_dir, 'mdf_reader/tests/data/125-704_1878-10_subset.imma')
    schema_new = 'imma1_d704'
    data_new = mdf_reader.read(data_file_path, data_model = schema_new)

    attributes = data_new.atts.copy()
    name_of_model = 'icoads_r3000_d704'

    cdm_dict = cdm.map_model(name_of_model, data_new.data, attributes, cdm_subset = None, log_level = 'INFO')

You can explore each table inside the ``cdm_dict``::

    cdm_dict['header']['data'].platform_sub_type.head()
    cdm_dict['observations-slp']['data'].observation_height_above_station_surface.head()
