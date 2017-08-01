Update portingdb data
---------------------

Setup (to be run just once and if needed)
*****************************************

To use the portingdb update scripts, you will need to install and configure the following:

#. Ensure that you run Fedora 26 or higher. The DNF plugin requires DNF-2 released in F26, due to incompatible CLI changes in API methods.

#. Put ``py3query.py`` in DNF plugin directory::
    
    sudo cp dnf-plugins/py3query.py /usr/lib/pythonX.X/site-packages/dnf-plugins/

   Note: replace ``pythonX.X`` with the Python version of your system.

#. Install the rawhide repo definitions::
    
    sudo dnf install fedora-repos-rawhide

#. Install ``python3-blessings`` which is an additional dependency for ``scripts/get-history.py``::

    sudo dnf install python3-blessings

Update and load data
********************

Follow the steps below to update pordingdb data:

#. Get the Python 3 porting status using ``py3query`` dnf plugin. Use ``-o`` option to write the output directly to ``fedora.json``::

    dnf-3 --disablerepo='*' --enablerepo=rawhide --enablerepo=rawhide-source py3query --refresh -o data/fedora.json

#. Get historical status data and update ``history.csv``::

    python3 -u scripts/get-history.py --update data/history.csv | tee history.csv
    mv history.csv data/history.csv

#. Load the newly generated data into the database::

    python3 -m portingdb -v --datadir=data/ load

   You might also view the data using the following command::

    sqlitebrowser portingdb.sqlite

#. Compare statuses of packages across two JSON files::

    python3 scripts/jsondiff.py <(git show HEAD:data/fedora.json) data/fedora.json

   At this point, make sure to take a closer look at the output of the command and for each change of package state find the appropriate actions in `Package state changes`_.

#. Commit changes, push to a fork and create a PR::

    git commit -avm 'Update Fedora data'

Package state changes
*********************

idle -> mispackaged
    A bug was opened:

* Check the bug, make sure it makes sense. Otherwise let someone know.

idle -> missing, mispackaged -> missing, released -> missing
    The package was dropped from Fedora or got rid of all dependencies:

* Find the package in the old version of portingdb, open spec or logs and look for the change that removed all python dependencies or retired the package.
* Note in the Etherpad or PR description in case of any of the above.
* If anything else happened, ask the team.

idle -> released, mispackaged -> released
    Someone has ported the package and it went to Fedora:

* Find the package in the newest version of portingdb and check the bug.
* Gather information to give a badge to the packager:

  * Find who is the packager: check the bug or the log of the spec file
  * Find the packager's fedora user name in pkdb
  * Put four pieces of info into the Etherpad or PR description:

    * username
    * full name
    * name of the project
    * name of the bug or commit (to put on the wiki)

missing -> released
    New Python 3 package was added to Fedora:

* Look through the package spec file to check if anything is important.
* Notify in the Etherpad or PR description about the new package.

missing -> idle
    The package got added to Fedora only with Python 2 version:

* Check the rpm list for Python 3 package.
* Look if the package is Python 3 compatible on upstream.
* Check package review request for any mention of Python 3 support.
* Check the upstream code for Python 3 support and if upstream is Python 3 compatible:

  * File a bug
  * Note in review request that Python 3 package was missed

relesed -> mispackaged
    There is an issue with Python 3 subpackage:

* Open the package in portingdb and check requirements for ``python3-`` subpackage:

  * If the subpackage is dragging py2 dependency in, check the latest change and find what caused it
  * File a bug

released -> idle
    There is an issue with the package:

* Let someone know.

Treat the "py3-only" state the same as "released".

If you encounter some other transition, let someone know and figure out a process!
