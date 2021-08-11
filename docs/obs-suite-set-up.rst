.. _obs-suite-set-up:

================================================
Observation suit set up and directories overview
================================================

1. Clone the latest version of the `glamod-marine-processing <https://github.com/glamod/glamod-marine-processing>`_ repository to a user workspace in the `JASMIN Cluster service <https://help.jasmin.ac.uk/category/158-getting-started>`_, most of the processing for the C3S project has taken place in the following directory ``/gws/smf/j04/c3s311a_lot2``, once there, you can type the following::

    $ mkdir work_user_dir
    $ cd work_user_dir
    $ git clone git@github.com:glamod/glamod-marine-processing.git
    $ cd glamod-marine-processing/
    $ ls
       config-suite  docs  metadata-suite  obs-suite  pre-processing  qc-suite

.. warning:: Before you do step 2, make sure to deactivate any **conda environment** that you might have in your JASMIN user directory by typing the command ``conda deactivate``, this in the case that you have previously installed your own conda outside of the tools that JASMIN provides i.e, jaspy. If you have never use your own conda, you can ignore this warning.

2. Set up the obs-suite python environment::

    $ cd obs-suite/pyenvs
    $ module load jaspy
    $ virtualenv --system-site-packages env0
    $ source env0/bin/activate
    (env0)$ pip install -r requirements_env0.txt

3. Install all additional modules, coded specifically for the C3s processing. Make sure to activated your python environment (env0) before cloning any repository. From these modules listed below, only two have been updated and documented, the `mdf_reader <https://glamod.github.io/mdf_reader_documentation/>`_ and `cdm_mapper <https://glamod.github.io/cdm_mapper_documentation/>`_::

    (env0)$ cd obs-suite/modules/python
    (env0)[python]$ git clone git@github.com:glamod/mdf_reader.git --branch master
    (env0)[python]$ git clone git@github.com:glamod/cdm-mapper.git --branch master --single-branch cdm
    (env0)[python]$ git clone git@github.com:glamod/metmetpy.git --branch v1.0
    (env0)[python]$ git clone git@github.com:glamod/pandas_operations.git --branch v1.2
    (env0)[python] $ ls
    cdm  mdf_reader  metmetpy  pandas_operations

.. note:: The attribute ``--branch`` is needed to clone a specific version of the code and ``--single-branch`` is only important for the **cdm_mapper** repository, since denotes how the modules are name in all the scripts of the code. The module name of the **cdm_mapper** repo inside the code is **cdm**. By cloning things with the ``--single-branch`` argument, you can only make changes to your local copy of the tool. You won't be able to push commits to the online repository if you use the single branch option. This set up is for users only; to develop and modify the upstream repositories you should clone the repositories without the ``--single-branch`` in a separate folder, make your changes and then commit. **But for running the C3S processing workflow the** ``--single-branch`` **is always needed to clone the cdm_mapper**.

4. Set the paths to the input ``data_directory``, ``configuration_directory``, ``environment_directory`` and ``code_directory``.

Under ``glamod-marine-processing/obs-suite/`` there are two .sh files (``setenv0.sh`` and ``setpaths.sh``) that contain paths that link the python environment, input and output data and the obs-suite code. The first file that you need to modify is the ``setpaths.sh`` to let the code be aware of where everything is::

        $ vim setpaths.sh

        #!/bin/bash
        export home_directory=/gws/nopw/j04/glamod_marine
        export home_directory_smf=/gws/smf/j04/c3s311a_lot2

        # YOU ONLY NEED TO MODIFY THIS LINE!! and only the part that says work_user_dir or something else in case that
        # you have store your glamod-marine-processing repository in another working space.
        export code_directory=$home_directory_smf/work_user_dir/glamod-marine-processing/obs-suite

        export config_directory=$home_directory_smf/code/marine_code/glamod-marine-config/obs-suite

        ## Below here you shouldn't need to modify anything!

        export pyTools_directory=$code_directory/modules/python
        export scripts_directory=$code_directory/scripts
        export lotus_scripts_directory=$code_directory/lotus_scripts
        export data_directory=$home_directory/data

        echo 'Data directory is '$data_directory
        echo 'Code directory is '$code_directory
        echo 'Config directory is '$config_directory
        # Create the scratch directory for the user
        export scratch_directory=/work/scratch-nopw/$(whoami)
        if [ ! -d $scratch_directory ]
        then
          echo "Creating user $(whoami) scratch directory $scratch_directory"
          mkdir $scratch_directory
        else
          echo "Scratch directory is $scratch_directory"
        fi

Most of the time the ``home_directory``, ``home_directory_smf`` and ``config_directory`` will not change unless there is a requirement in JASMIN to move working spaces. The only path that you should modify from the file above should be ``code_directory``.

After this you need to also change ``setenv0.sh`` to point to the environment created in steps 1-3::

        $ vim setenv0.sh

        pyEnvironment_directory=$code_directory/pyenvs/env0

        # Activate python environment and add jaspy3.7 or any other version of jaspy to LD_LIBRARY_PATH
          so that cartopy and other python tools can find the geos library
        source $pyEnvironment_directory/bin/activate
        export PYTHONPATH="$pyTools_directory:${PYTHONPATH}"

        # ONLY MODIFY THE LINE BELOW, to point out to the right jaspy module that you want to use
        export LD_LIBRARY_PATH=/apps/contrib/jaspy/miniconda_envs/jaspy3.7/m3-4.6.14/envs/jaspy3.7-m3-4.6.14-r20200606/lib/:$LD_LIBRARY_PATH
        echo "Python environment loaded from gws: $pyEnvironment_directory"

5. Deactivate your environment and check the connection by sourcing both .sh files under ``../obs-suite/``::

    $ deactivate env0
    $ module load jaspy
    $ source setpaths.sh
    $ source setenv0.sh

The commands above will activate the paths and the environment. You will have to do step 5 every time you logging to a new jasmin session. Use ``cd $code_directory`` or any dir name variable defined in ``setpaths.sh`` to navigate between all directories.

6. Explore the ``$data_directory``::

    $ cd $data_directory
    $ ls
        3          datasets           r092019      release_3.0  release_demo  user_manual
        dashboard  marine-user-guide  release_2.0  release_4.0  release_test

Unless working spaces in jasmin change, this directory is where you will always find the output of every release, and also where the ICOADS unprocessed data is stored (see ``/datasets/ICOADS_R3.0.0T/level0``). For each new release of ICOADS you will have the following directory tree::

        | release_4.0
        | ├── ICOADS_R3.0.0T
        | │   ├── level1a
        | │   ├── level1b
        | │   ├── level1c
        | │   ├── level1d
        | │   ├── level1e
        | │   ├── level1f
        | │   ├── level2
        | │   └── metoffice_qc
        | ├── ICOADS_R3.0.1T
        | ├── NOC_corrections
        | ├── wmo_publication_47

7. Explore the ``$config_directory``::

    $ cd $data_directory
    $ ls
    r092019-000000      release_3.0-000000  release_demo-000000
    release_2.0-000000  release_4.0-000000  release_test-000000

The configuration directory is where you define several ``.json`` and `.txt` files that will indicate to the ``obs-suite`` software the following:
    - What sources and decks we would be processing?
    - Do we need to apply a specific schema and mapper to a deck or a number of decks?
    - What periods of data are we going to process?

**Click on next to know more about how to set up a release and the configuration directory**
