.. _last-steps:

==========================
3. Commit and push changes
==========================

For deck 704, there is no need to commit and push the files created for the ``imma1_d704`` or ``icoads_3000_d704``, since all those files already exist in both tools main repositories (mdf_reader and cdm_mapper repos). But for the new CDM's that you will create, you will have to add these files to the Github main repositories.

- For the ``mdf_reader`` this is very easy::

    $ cd ../mdf_reader/
    $ git status
    $ git add .
    $ git status
    $ git commit -m "Added a new schema for deck No.xx"
    $ git push origin master

.. note:: Sometimes the remote branch can have the name **main or master** so make sure you check that in the Github repository online.

.. note:: Before pushing any change to the main repository make sure that you have maintainer access and you are allow to push/merge changes. If you don't have this, you can ask for access or keep a fork copy of the main repository in your github/gitlab account. If you use your own copy, make sure you clone that copy in the obs-suit framework, when you want to test and run your new CDM.

- For the ``cdm_mapper`` committing changes to the main repository **is different**. Because the python module name in the **obs-suit** code differs from the repository name. Module name = **cdm**, Repository name = **cdm_mapper**. This means that when we clone the cdm_mapper as a module, we use the attribute ``--single-branch``. This does not allow direct commits to the main repository. Therefore, to commit changes to this repository you need to clone again the mapper repo in the normal way::

    $ cd ../to_your_designated_folder_where_you_installed_mdf_reader_and_cdm/
    $ git clone git@github.com:glamod/cdm-mapper.git --branch master

- Now copy from the cdm library, the new added mapper to the cdm_mapper library folder and commit::

    $ cp -r ../cdm/lib/mappings/icoads_3000_dXXX ../cdm_mapper/lib/mappings/.
    $ cd ./cdm_mapper/
    $ git status
    $ git add .
    $ git status
    $ git commit -m "Added a new mapper for deck No.xx"
    $ git push origin main

=================================================
4. Clone the latest version of the modified tools
=================================================

- Once you have commit your changes to the online version of both repositories, you can go back to JASMIN and pull the latest changes. We recommend that in the JASMIN obs-suit installation folder, you always delete old versions of the **mdf_reader** and **cdm** folders under ``obs-suit/modules/python`` before cloning again the latest changes::

    $ cd ../obs-suit/modules/python
    $ rm -r mdf_reader
    $ rm -r cdm

- Before cloning again the modules, you should remember to activate the python environment by::

    $ cd ../obs-suit
    $ module load jaspy
    $ source setpaths.sh
    $ source setenv0.sh

- Clone the repositories again making sure your environment is activated::

    (env0)$ cd obs-suit/modules/python
    (env0)[python]$ git clone git@github.com:glamod/mdf_reader.git --branch master
    (env0)[python]$ git clone git@github.com:glamod/cdm-mapper.git --branch master --single-branch cdm
    (env0)[python] $ ls
    cdm  mdf_reader  metmetpy  pandas_operations

- Check in the libraries of both repositories if your new CDM is there. Test your installation and environment by importing the tools in a python console.

=======================================
5. Run level1a for that new sid-dck CDM
=======================================

To run level1a for an specific deck, you need to modify the ``level1a.json`` file located at the ``$config_dir``. Below you will see an example of how to set up this run for deck 704:

- ``vim $config_directory/release_demo-000000/ICOADS_R3.0.0T/level1a.json``::

    {
        "job_memo_mb": 16000,
        "job_time_hr": "03",
        "job_time_min": "30",
        "data_model": "imma1",
        "read_sections": ["core","c1","c98"],
        "filter_reports_by":
        {
          "c1.PT": ["0","1","2","3","4","5"]
        },
        "cdm_map": "icoads_r3000",
        "source_pattern":"IMMA1_R3.0.0T_????-??",
        "125-704":
            {
            "job_memo_mb": 4000,
            "job_time_hr": "01",
            "job_time_min": "30",
            "data_model": "imma1_d704",
            "read_sections": [
                "core",
                "c1",
                "c98",
                "c99_journal"
            ],
            "filter_reports_by":
            {
             "c1.PT": ["0","1","2","3","4","5"]
            },
            "cdm_map": "icoads_r3000_d704",
            "source_pattern":"IMMA1_R3.0.0T_????-??"
        }
    }

- Make sure you have the correct information for the ``source_deck_list.txt`` and ``source_deck_periods.json``. To learn how to set up these two files and run level1a, please referred again to the :ref:`release-workflow` documentation.
