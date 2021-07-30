.. _built-new-schema:

====================================
1. Build and add a mdf_reader schema
====================================

Detail instructions on how to build and add a new schema for any type of data can be found in the `mdf_reader docs <https://glamod.github.io/mdf_reader_documentation/how-to-build-a-data-model.html>`_. However, the process to create a **schema** for a data collection from ICOADS can be simplified by copying an **already built in** schema i.e., ``imma1``. This can be adapted for a specific deck, by just modifying the last section of the ``imma1.json`` file (the ``c99`` section) and adding `code_tables <https://glamod.github.io/mdf_reader_documentation/data-models.html#code-tables>`_ needed to decode certain information. The ``c99`` section contains all the original collection supplemental metadata. The modification of this section will allowed you to read and map new metadata information to the C3S CDM format.

1.1 Set up your script
----------------------
After activating the correct python environment, you can start a new python console and create the following script by typing the commands below. Don't forget to change your working directory path first::

    import os
    import sys
    working_dir = '/to_your_designated_folder_where_you_installed_mdf_reader_and_cdm/'
    sys.path.append(working_dir)
    import mdf_reader
    import json

To get familiar with the reader tool you can read a raw `.imma` file from dck 704 as an example, for a subset of the data collected in 1878/10::

    schema = 'imma1'
    data_file_path = os.path.join(working_dir, 'mdf_reader/tests/data/125-704_1878-10_subset.imma')
    data_raw = mdf_reader.read(data_file_path, data_model = schema)

Now you can close your script, but don't forget to save it!

1.2 Copy files from a pre-existent data model
---------------------------------------------

To construct the schema for deck 704 you can copy a the default ``imma1`` schema and modify the ``c99`` section. Open a shell and do the following::

    $ cd ../mdf_reader/data_models/lib
    $ cp -r imma1 imma1_d704
    $ cd imma1_d704
    $ mv imma1.json imma1_d704.json

.. warning::
    It is important that you change the ``imma1.json`` file name to the corresponding deck number (in this case ``imma1_d704.json``). The main folder of the schema and the ``.json`` file need to have the same name, so the mdf_reader tool is able to recognize it .

1.3 Modify the ``imma1_d704.json`` file
---------------------------------------

- Now in the editor of your choice modify ``imma1_d704.json`` following the instructions below:

    - In the `header <https://glamod.github.io/mdf_reader_documentation/how-to-build-a-data-model.html#schema-header-block>`_ section::

        "header": {
        "parsing_order": [{"s": ["core"]},{"o": ["c1","c5","c6","c7","c8","c9","c95","c96","c97", "c98","c99"]}]
        }
    - Replace the text by::

        "header": {
            "parsing_order": [
              {"s": ["core"]},
              {"o": ["c1","c5","c6","c7","c8","c9","c95","c96","c97","c98"]},
              {"s": ["c99_sentinal","c99_journal","c99_voyage","c99_daily"]},
              {"e": ["c99_data4","c99_data5"]}
            ]
        }
    Here we are adding new sub-sections to the **header** to subdivide the ``c99`` string into its corresponding sub-sections.
    We found the information of how this string is subdivided by searching for this deck on the `ICOADS website <https://icoads.noaa.gov/e-doc/other/transpec/usmmj/mmj_transpec>`_.

    - Now we will delete the following text (~ from line 2214 onwards)::

            "c99": {
            "header": {"sentinal": "99 0", "disable_read": true}
             }

    - Each sub-section of the ``c99`` string declared at the header, is composed by a group of delimited elements that are decoded according to this `documentation from ICOADS <https://icoads.noaa.gov/e-doc/other/transpec/usmmj/mmjdoc.pdf>`_. The information from the ICOADS website has been translated to a ``.json`` python dictionary. You can now replace the above ``c99`` text by copying the following subsections below::

            "c99_sentinal": {
                "header": {"sentinal": "99 0 ", "length": 5, "field_layout":"fixed_width"},
                "elements": {
                  "ATTI": {
                      "description": "attm ID",
                      "field_length": 2,
                      "column_type": "str",
                      "ignore":false
                  },
                  "ATTL": {
                      "description": "attm length",
                      "field_length": 2,
                      "column_type": "int8",
                      "valid_max": 0,
                      "valid_min": 0,
                      "ignore":false
                  },
                  "BLK": {
                      "description": "blank space",
                      "field_length": 1,
                      "column_type": "object",
                      "ignore": false,
                      "disable_white_strip": true
                  }
                }
            },
            "c99_journal": {
                "header": {"sentinal": "1", "field_layout":"fixed_width","length": 117},
                "elements": {
                  "sentinal":{
                      "description": "Journal header record identifier",
                      "field_length": 1,
                      "column_type": "str"
                  },
                  "reel_no":{
                      "description": "Microfilm reel number. See if we want the zero padding or not...",
                      "field_length": 3,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "journal_no": {
                      "description": "Marine Meteorological Journal number,right justified, zero padded",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "frame_no":{
                      "description": "microfilm frame number found at number the top of the frame containing the Journal # and name of ship",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "ship_name": {
                      "description": "Ship name, left justified",
                      "field_length": 26,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "journal_ed": {
                      "description": "Last two digits of year of issue",
                      "field_length": 2,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "rig": {
                      "description": "Rig (type of ship) as appeared at beginning of journal",
                      "field_length": 2,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.RIG",
                      "LMR6": false
                  },
                  "ship_material": {
                      "description": "Construction material",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.MAT",
                      "LMR6": false
                  },
                  "vessel_type": {
                      "description": "Type of vessel",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.VESS",
                      "LMR6": false
                  },
                  "vessel_length": {
                      "description": "Length of vessel in feet rounded to nearest whole foot",
                      "field_length": 3,
                      "column_type": "Int16",
                      "LMR6": false
                  },
                  "vessel_beam": {
                      "description": "Width of vessel in feet rounded to nearest whole foot",
                      "field_length": 2,
                      "column_type": "Int16",
                      "LMR6": false
                  },
                  "commander": {
                      "description": "Commander as appeared in Journal, left justified",
                      "field_length": 15,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "country": {
                      "description": "Nation of registry",
                      "field_length": 2,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.NAT",
                      "LMR6": false
                  },
                  "screw_paddle": {
                      "description": "Screw or paddle",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.SP",
                      "LMR6": false
                  },
                  "hold_depth": {
                      "description": "Rounded to the nearest whole foot",
                      "field_length": 2,
                      "column_type": "Int16",
                      "LMR6": false
                  },
                  "tonnage": {
                      "description": "Rounded to nearest whole ton",
                      "field_length": 4,
                      "column_type": "Int16",
                      "LMR6": false
                  },
                  "baro_type": {
                      "description": "Barometer type. Add code table",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.BAROT",
                      "LMR6": true
                  },
                  "baro_height": {
                      "description": "Barometer height above sea level in feet",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": false
                  },
                  "baro_cdate": {
                      "description": "Date barometer last compared to standard (ddmmyyyy)",
                      "field_length": 8,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "baro_loc": {
                      "description": "Barometer location description",
                      "field_length": 25,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "baro_units": {
                      "description": "Barometer units",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.BAROU",
                      "LMR6": false
                  },
                  "baro_cor": {
                      "description": "Journal header record.Do something with this, see digitization format doc",
                      "field_length": 6,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "thermo_mount": {
                      "description": "Are thermos mounted as recommended in journal",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.THERMOM",
                      "LMR6": false
                  },
                  "SST_I": {
                      "description": "Sea surface temperature method",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.SSTM",
                      "LMR6": true
                  }
                }
            },
            "c99_voyage": {
                "header": {"sentinal": "2", "length": 52,"field_layout":"fixed_width"},
                "elements": {
                  "sentinal": {
                      "description": "Voyage header record identifier",
                      "field_length": 1,
                      "column_type": "str"
                  },
                  "reel_no":{
                      "description": "Microfilm reel number. See if we want the zero padding or not...",
                      "field_length": 3,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "journal_no": {
                      "description": "Marine Meteorological Journal number,right justified, zero padded",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "frame_start": {
                      "description": "As starting frame number in the voyage header record: ensures connection between header info and data",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "from_city": {
                      "description": "Departure port as it appears on the form",
                      "field_length": 20,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "to_city": {
                      "description": "Destination port as it appears on the form",
                      "field_length": 20,
                      "column_type": "str",
                      "LMR6": false
                  }
                }
            },
            "c99_daily": {
                "header": {"sentinal": "3", "field_layout":"fixed_width","length": 61},
                "elements": {
                  "sentinal": {
                      "description": "Daily information record identifier",
                      "field_length": 1,
                      "column_type": "str"
                  },
                  "reel_no":{
                      "description": "Microfilm reel number. See if we want the zero padding or not...",
                      "field_length": 3,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "journal_no": {
                      "description": "Marine Meteorological Journal number,right justified, zero padded",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "frame_start": {
                      "description": "As starting frame number in the voyage header record: ensures connection between header info and data",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "frame": {
                      "description": "Frame number at the top of each frame from which the daily inforation was extracted",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "year": {
                      "description": "As indicated in the form",
                      "field_length": 4,
                      "column_type": "Int16"
                  },
                  "month": {
                      "description": "As indicated in the form",
                      "field_length": 2,
                      "column_type": "Int8"
                  },
                  "day": {
                      "description": "As indicated in the form",
                      "field_length": 2,
                      "column_type": "Int8"
                  },
                  "distance": {
                      "description": "Distance run by log since prededing run by log noon. Knots to tenths (tenth position blank if not reported)",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": false
                  },
                  "lat_deg_an": {
                      "description": "latitude by account at noon, degrees",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lat_min_an": {
                      "description": "latitude by account at noon, minutes",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lat_hemis_an": {
                      "description": "latitude by account at noon, hemisphere",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "lon_deg_an": {
                      "description": "longitude by account at noon, degrees",
                      "field_length": 3,
                      "column_type": "Int16",
                      "LMR6": true
                  },
                  "lon_min_an": {
                      "description": "longitude by account at noon, minutes",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lon_hemis_an": {
                      "description": "longitude by account at noon, hemisphere",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "lat_deg_on": {
                      "description": "latitude by observation at noon, degrees",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lat_min_on": {
                      "description": "latitude by observation at noon, minutes",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lat_hemis_on": {
                      "description": "latitude by observation at noon, hemisphere",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "lon_deg_of": {
                      "description": "longitude by chronometer from forenoon observation, degrees",
                      "field_length": 3,
                      "column_type": "Int16",
                      "LMR6": true
                  },
                  "lon_min_of": {
                      "description": "longitude by chronometer from forenoon observation, minutes",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "lon_hemis_of": {
                      "description": "longitude by chronometer from forenoon observation, hemisphere",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "current_speed": {
                      "description": "Current speed in the past 24 hours. To knots hundreths",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.01,
                      "LMR6": false
                  },
                  "current_direction": {
                      "description": "Direction towards which the currents are moving. Code table? not in document, probably as other directions....",
                      "field_length": 7,
                      "column_type": "str",
                      "LMR6": false
                  }
                }
            },
            "c99_data4": {
                  "header": {"sentinal" : "4", "field_layout":"fixed_width","length": 97},
                  "elements": {
                    "sentinal": {
                        "description": "Data information record type identifier",
                        "field_length": 1,
                        "column_type": "str"
                    },
                    "reel_no": {
                        "description":"The number from the microfilm roll",
                        "field_length": 3,
                        "column_type": "str",
                        "LMR6": true
                    },
                    "journal_no": {
                        "description": "Meteorological Journal Number, right justified, zero filled",
                        "field_length": 4,
                        "column_type": "str",
                        "LMR6": true
                    },
                    "frame_start": {
                        "description": "As starting frame number in the voyage header record: ensures connection between header info and data",
                        "field_length": 4,
                        "column_type": "str",
                        "LMR6": false
                    },
                    "frame": {
                        "description": "Frame number at the top of each frame from which the daily inforation was extracted",
                        "field_length": 4,
                        "column_type": "str",
                        "LMR6": false
                    },
                    "year": {
                        "description": "Year",
                        "field_length": 4,
                        "column_type": "Int16",
                        "LMR6": true
                    },
                    "month": {
                        "description": "Month",
                        "field_length": 2,
                        "column_type": "Int8",
                        "LMR6": true
                    },
                    "day": {
                        "description": "Day",
                        "field_length": 2,
                        "column_type": "Int8",
                        "LMR6": true
                    },
                    "time_ind": {
                        "description": "Time indicator",
                        "field_length": 1,
                        "column_type": "key",
                        "codetable": "ICOADS.C99.TI",
                        "LMR6": true
                    },
                    "hour": {
                        "description": "Hour, right justified, zero filled",
                        "field_length": 2,
                        "column_type": "Int8",
                        "LMR6": true
                    },
                    "ship_speed": {
                        "description": "Ship speed knots, to tenths",
                        "field_length": 3,
                        "column_type": "float16",
                        "scale": 0.1,
                        "LMR6": true
                    },
                    "compass_ind": {
                        "description": "Compass indicator. Reflects: course steered, compass correction, Leeway, ships true course",
                        "field_length": 1,
                        "column_type": "key",
                        "codetable": "ICOADS.C99.CI",
                        "LMR6": false
                    },
                    "ship_course_compass": {
                        "description": "Ship course steered by compass. See digitization format for translation",
                        "field_length": 7,
                        "column_type": "str",
                        "LMR6": false
                    },
                    "compass_correction": {
                        "description": "Correction in points or degrees, as indicated in compass_ind. LMR6 false? see ref Jackson et. al. 2000 in translation info",
                        "field_length": 2,
                        "column_type": "Int8",
                        "LMR6": false
                    },
                    "ship_course_true": {
                        "description": "Ship course true. Same rules as course steered by compass",
                        "field_length": 7,
                        "column_type": "str",
                        "LMR6": true
                    },
                    "wind_dir_mag": {
                        "description": "Mean wind direction - magnetic. Same rules as course steered by compass",
                        "field_length": 7,
                        "column_type": "str",
                        "LMR6": true
                    },
                    "wind_dir_true": {
                        "description": "Wind direction - true. Same rules as course steered by compass",
                        "field_length": 7,
                        "column_type": "str",
                        "LMR6": true
                    },
                    "wind_force": {
                        "description": "Mean Beaufort force",
                        "field_length": 2,
                        "column_type": "key",
                        "codetable": "ICOADS.C99.BEAU",
                        "LMR6": true
                    },
                    "barometer": {
                        "description": "Barometer in inches or millimetres. Left|right justified to confirm",
                        "field_length": 4,
                        "column_type": "str",
                        "disable_white_strip": true,
                        "LMR6": true
                    },
                    "temp_ind": {
                        "description": "Temperature indicator",
                        "field_length": 1,
                        "column_type": "key",
                        "codetable": "ICOADS.C99.TEMPI",
                        "LMR6": true
                    },
                    "attached_thermometer": {
                        "description": "Attached thermometer. It's integer values from tens to hundreds...",
                        "field_length": 4,
                        "column_type": "float16",
                        "scale": 0.1,
                        "LMR6": true
                    },
                    "air_temperature": {
                        "description": "Air temperature. Units according to temperature indicator",
                        "field_length": 4,
                        "column_type": "float16",
                        "scale": 0.1,
                        "LMR6": true
                    },
                    "wet_bulb_temperature": {
                        "description": "Wet bulb temperature. Units according to temperature indicator",
                        "field_length": 4,
                        "column_type": "float16",
                        "scale": 0.1,
                        "LMR6": true
                    },
                    "sea_temperature": {
                        "description": "Sea surface temperature",
                        "field_length": 4,
                        "column_type":  "float16",
                        "scale": 0.1,
                        "LMR6": true
                    },
                    "present_weather": {
                        "description": "Present weather, left justified, blank filled. Split in 5?",
                        "field_length": 5,
                        "column_type": "str",
                        "LMR6": false
                    },
                    "clouds": {
                        "description": "Forms of clouds by symbols",
                        "field_length": 2,
                        "column_type": "str",
                        "LMR6": false
                    },
                    "sky_clear": {
                        "description": "Clear sky in tenths",
                        "field_length": 2,
                        "column_type": "Int8",
                        "LMR6": true
                    },
                    "sea_state": {
                        "description": "State of the sea, left justified, blank filled. Split in 4?",
                        "field_length": 4,
                        "column_type": "str",
                        "LMR6": false
                    }
                  }
              },
            "c99_data5": {
                "header": {"sentinal" : "5", "field_layout":"fixed_width","length": 103},
                "elements": {
                  "sentinal": {
                      "description": "Data information record type identifier",
                      "field_length": 1,
                      "column_type": "str"
                  },
                  "reel_no": {
                      "description":"The number from the microfilm roll",
                      "field_length": 3,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "journal_no": {
                      "description": "Meteorological Journal Number, right justified, zero filled",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "frame_start": {
                      "description": "As starting frame number in the voyage header record: ensures connection between header info and data",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "frame": {
                      "description": "Frame number at the top of each frame from which the daily inforation was extracted",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "year": {
                      "description": "Year",
                      "field_length": 4,
                      "column_type": "Int16",
                      "LMR6": true
                  },
                  "month": {
                      "description": "Month",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "day": {
                      "description": "Day",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "time_ind": {
                      "description": "Time indicator",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.TI",
                      "LMR6": false
                  },
                  "hour": {
                      "description": "Hour, right justified, zero filled",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "ship_speed": {
                      "description": "Ship speed knots, to tenths",
                      "field_length": 3,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "compass_ind": {
                      "description": "Compass indicator",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.CI",
                      "LMR6": false
                  },
                  "ship_course_compass": {
                      "description": "Ship course steered by compass",
                      "field_length": 7,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "blank": {
                      "description": "blank fields",
                      "field_length": 2,
                      "column_type": "str"
                  },
                  "ship_course_true": {
                      "description": "Ship course true",
                      "field_length": 7,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "wind_dir_mag": {
                      "description": "Mean wind direction - magnetic",
                      "field_length": 7,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "wind_dir_true": {
                      "description": "Wind direction - true",
                      "field_length": 7,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "wind_force": {
                      "description": "Mean Beaufort force. Add code table",
                      "field_length": 2,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.BEAU",
                      "LMR6": true
                  },
                  "barometer": {
                      "description": "Barometer in inches or millimetres. Left|right justified to confirm",
                      "field_length": 4,
                      "column_type": "str",
                      "disable_white_strip": true,
                      "LMR6": true
                  },
                  "temp_ind": {
                      "description": "Temperature indicator",
                      "field_length": 1,
                      "column_type": "key",
                      "codetable": "ICOADS.C99.TEMPI",
                      "LMR6": true
                  },
                  "attached_thermometer": {
                      "description": "Attached thermometer. It's integer values from tens to hundreds...",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "air_temperature": {
                      "description": "Air temperature. Units according to temperature indicator",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "wet_bulb_temperature": {
                      "description": "Wet bulb temperature. Units according to temperature indicator",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "sea_temperature": {
                      "description": "Sea surface temperature",
                      "field_length": 4,
                      "column_type":  "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "present_weather": {
                      "description": "Present weather, left justified, blank filled. Split in 5?",
                      "field_length": 5,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "clouds": {
                      "description": "Forms of clouds by symbols",
                      "field_length": 2,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "sky_clear": {
                      "description": "Clear sky in tenths",
                      "field_length": 2,
                      "column_type": "Int8",
                      "LMR6": true
                  },
                  "sea_state": {
                      "description": "State of the sea, left justified, blank filled. Split in 4?",
                      "field_length": 4,
                      "column_type": "str",
                      "LMR6": false
                  },
                  "compass_correction_ind": {
                      "description": "Compass correction indicator",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  },
                  "compass_correction": {
                      "description": "compass correction",
                      "field_length": 4,
                      "column_type": "float16",
                      "scale": 0.1,
                      "LMR6": true
                  },
                  "compass_correction_dir": {
                      "description": "Direction of correction (of compass...)",
                      "field_length": 1,
                      "column_type": "str",
                      "LMR6": true
                  }
                }
            }
.. note:: For future decks you will have to type all this information yourself. Basically you will need to translate all the supplemental data information for that source&deck from the ICOADS website to the ``.json`` file. The goal is to successfully separate the ``c99`` string into different sections of data and elements. You will have to add each element to the ``schema.json`` and make sure each element has the correct `descriptors <https://glamod.github.io/mdf_reader_documentation/how-to-build-a-data-model.html#>`_.

- Save all modifications done to ``imma1_d704.json``, make sure the ``.json`` file is valid. You can copy the entire content and check via an `online tool <https://jsonlint.com/>`_ or just make sure you use a json editor.

1.4 Add the ``code_tables``
---------------------------

Now we will **add** code tables needed by some elements to decode information. Add the following files to the schema ``code_tables`` folder ``../mdf_reader/data_models/lib/imma1_d704/code_tables/``. This part can be avoided by copying from the **mdf_reader main repository** all the code tables for this deck, if you don't want to generate the files yourself:

    - ``$ vim ICOADS.C99.BAROT.json``::

        {
            "1":"aneroid",
            "2":"mercurial"
        }
    - ``$ vim ICOADS.C99.BAROU.json``::

        {
            "1":"inches",
            "2":"millimeters",
            "3":"millibars",
            "4":"unable to determine",
            "5":"Paris inches"
        }

    - ``$ vim ICOADS.C99.BEAU.json``::

        {
            "01":"Light air",
            "02":"Light breeze",
            "03":"Gentle breeze",
            "04":"Moderate breeze",
            "05":"Fresh breeze",
            "06":"Strong breeze",
            "07":"High wind,moderate gale,near gale",
            "08":"Gale,fresh gale",
            "09":"Strong, severe gale",
            "10":"Storm,whole gale",
            "11":"Violent storm",
            "12":"Hurricane force"
        }

    - ``$ vim ICOADS.C99.CI.json``::

        {
            "1":"points",
            "2":"degrees",
            "3":"not indicated"
        }

    - ``$ vim ICOADS.C99.MAT.json``::

        {
            "1":"wood",
            "2":"iron",
            "3":"composite",
            "4":"not identified"
        }

    - ``$ vim ICOADS.C99.NAT.json``::

        {
            "01":"American",
            "02":"British",
            "03":"Chinese",
            "04":"French",
            "05":"Austrian",
            "06":"Dutch",
            "07":"Russian",
            "08":"German",
            "09":"Canadian",
            "10":"Belgian",
            "11":"Danish",
            "12":"Italian",
            "13":"Norwegian",
            "14":"Nova Scotian",
            "15":"Portuguese",
            "16":"Scottish",
            "17":"Swedish",
            "21":"Singaporean",
            "99":"undefined",
            "20":"undefined"
        }

    - ``$ vim ICOADS.C99.RIG.json``::

        {
            "01":"ship",
            "02":"bark or barque",
            "03":"barkentine or barquentine",
            "04":"brigantine",
            "05":"schooner",
            "06":"frigate",
            "99":"not identified"
        }

    - ``$ vim ICOADS.C99.SP.json``::

        {
            "1":"screw",
            "2":"paddle",
            "3":"not identified"
        }

    - ``$ vim ICOADS.C99.SSTM.json``::

        {
        "1":"bucket",
        "2":"intake",
        "3":"as directed in the journals instructions",
        "4":"not indicated"
        }

    - ``$ vim ICOADS.C99.TEMPI.json``::

        {
            "1":"Fahrenheit",
            "2":"Centigrade",
            "3":"Attached thermometer is Fahrenheit. Dry bulb, wet bulb and water temperature are Centigrade",
            "4":"Attached thermometer is Centigrade. Dry bulb, wet bulb and water temperature are Fahrenheit",
            "5":"Water temperature is Centigrade with other temperatures in Fahrenheit",
            "6":"Dry bulb and wet bulb are Centigrade with other temperatures in Fahrenheit"
        }

    - ``$ vim ICOADS.C99.THERMOM.json``::

        {
            "1":"mounted as recommended in journal introduction",
            "2":"not mounted as recommended in journal introduction",
            "3":"not indicated",
            "4":"not indicated"
        }

    - ``$ vim ICOADS.C99.TI.json``::

        {
            "1":"AM (local)",
            "2":"PM (local)"
        }

    - ``$ vim ICOADS.C99.VESS.json``::

        {
        "1":"sailing ship",
        "2":"steamer",
        "3":"not identified",
        "4":"not identified"
        }

.. note:: Each ``code_table`` above was created by reading and translating the information found in the ICOADS website for this particular deck. The name of the file corresponds to the section.variable that need such decoding.

1.5 Re-start your python environment
------------------------------------
- This can be done simply by closing and re-starting your shell or deactivating your virtual environment and activating it again.

1.6 Add to your python script the new schema
--------------------------------------------

Add the following lines::

    import os
    import sys
    working_dir = '/to_your_designated_folder_where_you_installed_mdf_reader_and_cdm/'
    sys.path.append(working_dir)
    import mdf_reader
    import json

    schema = 'imma1'
    data_file_path = os.path.join(working_dir, 'mdf_reader/tests/data/125-704_1878-10_subset.imma')
    data_raw = mdf_reader.read(data_file_path, data_model = schema)

    print('Now we apply the new schema')
    print('---------------------------')

    # ADD THIS NEW LINES!
    schema_new = 'imma1_d704'
    data_new = mdf_reader.read(data_file_path, data_model = schema_new)

.. note:: The metadata information of several decks in ICOADS has been documented in a word template as part of the C3S project to avoid users to find it themselves in the ICOADS website, this information can be found in the following url: **PENDING ADD URL to ICOADS structure folder**.