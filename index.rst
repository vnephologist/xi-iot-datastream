.. title:: Nutanix Python API Lab

.. toctree::
  :maxdepth: 2
  :caption:     Required Labs
  :name: _req-labs
  :hidden:

  .. example/index
  contents/lab

.. _getting_started:

What We Are Doing
#################

.. raw:: html

  <strong><font color="red">Do not start any labs before being told to do so by your Presenter.</font></strong>
  
The Nutanix Python API Lab will cover a couple of key points.

- Creation of a simple Python Flask web application.
- Creation of a single basic view to display cluster details for the user.
- A backend model to talk to the Nutanix APIs.
- JavaScript to create the interface between the front- and back-end parts of the application.

.. _requirements:

What We Aren't Doing
####################

This lab is not intended as a guide that can be used to learn Python development.  While the copy & paste steps will allow you to create a working application, previous experience with Python will aid you in understanding what each section does.

However, the lab will include links to valuable explanation and learning resources that can be used at any time for more information on each section.  For example, the general structure of this application is almost identical to the one provided by the official Python Flask tutorial and will often link to resources there.

Requirements
############

To successfully complete this lab, you will need an environment that meets the following specifications.

- Previous experience with Python is recommended but not strictly mandatory
- An installation of Python 3.6 or later.  For OS-specific information, please see the next section.
- Python `pip` for Python 3.6.
- Python Flask.  On **most** systems this should be case of running `pip3 install flask`.   On Windows, `pip3.exe` is in the `Scripts` folder within the Python install location.
- The text editor of your choice.  A good suggestion is Visual Studio Code_ as it is free and supports Python development via plugin.
- cURL (included in OS X and Linux).
- cURL (for Windows - see below).

.. _code: https://code.visualstudio.com/

Python 3.6 on OS X
------------------

- Install Python 3.6 on OS X by downloading the OS X installation package_.

.. _package: https://www.python.org/downloads/mac-osx/

Python 3.6 on Ubuntu 18.04
--------------------------

From the terminal, the following commands can be used to install Python 3.6:

.. code-block:: bash

    sudo apt-get -y update
    sudo apt-get -y install python3-dev python3-pip
    sudo apt-get -y install python3-venv
    sudo apt-get -y install python3-setuptools

Python 3.6 on CentOS 7
----------------------

From the terminal, the following commands can be used to install Python 3.6:

.. code-block:: bash

    sudo yum -y update
    sudo yum -y install epel-release
    sudo yum -y install python36
    python3.6 -m ensurepip
    sudo yum -y install python36-setuptools

Python 3.6 and cURL on Windows
------------------------------

- Install Python 3.6 by downloading the Python 3.6 installer_.
- Install cURL from the cURL website_.

.. _installer: https://www.python.org/downloads/release/python-360/
.. _website: https://curl.haxx.se/windows/

Note that cURL is not required to create the demo app.  cURL command samples are provided throughout the lab and may be used for reference at any time.  This is due to its cross-platform nature vs supporting platform-specific commands (e.g. PowerShell).

**Note re Windows systems:** As at January 2019, a **default** installation of Python 3.6 will be installed in the following folder:

.. code-block:: bash

    C:\Users\<username>\AppData\Local\Programs\Python\Python36

.. _optional_components:

Optional Components
###################

In addition to the requirement components above, the following things are "nice to have".  They are not mandatory for these labs.

- A Github account.  This can be created by signing up directly through GitHub_.
- The GitHub Desktop_ application (available for Windows and Mac only)
- Postman_, one of the most popular API testing tools available.

.. _GitHub: https://github.com/
.. _Desktop: https://desktop.github.com/
.. _Postman: https://www.getpostman.com/

.. _cluster_details:

Cluster Details
###############

In a presenter-led environment you will be using a shared Nutanix cluster.  Please use this cluster when carrying out your cURL and application testing.

Access details for the shared cluster will be provided by your presenter.