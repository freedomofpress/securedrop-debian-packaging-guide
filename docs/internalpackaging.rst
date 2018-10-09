Making your project ready for using virtualenv in Debian package
=================================================================

For this part of the tutorial, we will add a new dependency ``cryptography`` to
our example project, and we will also import the library inside of our source
code.


Install computepipfilehash tool
--------------------------------

::

    $ pip3 install computepipfilehash --user -U


The above command will install `computepipfilehash
<https://github.com/kushaldas/computepipfilehash>`_ tool. Use ``0.0.2``
version for this guide.


Change the example code
------------------------

First, we will move into the source directory and checkout a different
branch. Ensure that ``Pipfile.lock`` exists in the current working directory
and then:

::

    cd ~/code/whosaysthat/
    git checkout fancyrelease
    computepipfilehash > requirements-build.txt

The final command above creates a ``requirements-build.txt`` file for the
dependencies. We will use this file to build the wheels locally. Next, we should
move into our development environment and create an empty directory.

.. note:: We will sync source tarballs and binary wheels to the localwheels directory here.

Next, we will download missing source tarballs from PyPI.

::

    mkdir localwheels
    sd-downloadsources


Then, update the ``requirements-build.txt`` file with the hashes from the existing wheels.

::

    computepipfilehash --update-hashes


Finally, we can build the missing binary wheels from the sources.

::

    sd-buildwheels



The above command will build the wheels in the ``./localwheels/`` directory.
But, this will fail as some development header files are missing. We should
install all external C level dependencies from the Debian repository itself.
After installing the packages, we should retry to build the wheels again.

.. note:: Remember to commit the ``requirements-build.txt`` file to the git repo. This
          will help others to build using the same source tarballs.


::

    sudo apt-get install libssl-dev libffi-dev
    sd-buildwheels
    ls ./localwheels/
    asn1crypto-0.24.0-py3-none-any.whl
    certifi-2018.8.24-py2.py3-none-any.whl
    cffi-1.11.5-cp35-cp35m-linux_x86_64.whl
    chardet-3.0.4-py2.py3-none-any.whl
    cryptography-2.3.1-cp35-cp35m-linux_x86_64.whl
    idna-2.7-py2.py3-none-any.whl
    pycparser-2.19-py2.py3-none-any.whl
    requests-2.19.1-py2.py3-none-any.whl
    six-1.11.0-py2.py3-none-any.whl
    urllib3-1.23-py2.py3-none-any.whl


Create the requirements.txt file for our wheels
------------------------------------------------

As the next step, we will create the final ``requirements.txt`` file which will contain the details
of the wheels including the hashes.

::

    computepipfilehash --wheel-hashes > requirements.txt


Sync the local wheels into a central storage
----------------------------------------------


.. note:: Here we will have to figure out the steps to move the wheels to a central location.



Work on Debian packaging
-------------------------

Then, we will create a new source tarball for our project and also copy the wheels.

::

    
    python3 setup.py sdist
    cp dist/whosaysthat-0.0.2.tar.gz ~/packaging/
    cd ~/packaging/
    tar -xvf whosaysthat-0.0.2.tar.gz
    cd whosaysthat-0.0.2/
    cp -r ~/code/whosaysthat/localwheels .


Now, we will create the files required for our packaging manually, including the
``debian`` directory. We will also install ``dh-virtualenv`` package.

::

    $ mkdir debian
    $ sudo apt-get install dh-virtualenv


Create the compatibility file
------------------------------

::

    $ echo "9" > debian/compat


Create the control file
------------------------

Add the following text to the ``debian/control`` file.

::

    Source: whosaysthat
    Section: unknown
    Priority: optional
    Maintainer: Kushal Das <kushal@freedom.press>
    Build-Depends: debhelper (>= 9), dh-python, python3-all, python3-setuptools, dh-virtualenv
    Standards-Version: 3.9.8
    Homepage: https://github.com/freedomofpress/yourpackage
    X-Python3-Version: >= 3.5

    Package: whosaysthat
    Architecture: all
    Depends: ${python3:Depends}, ${misc:Depends}
    Description: This is our example tool
     This package installs the library for Python 3.

If we know any library we are dependent on (written in C), we should explicitly mention that in the
``Depends:`` line above.


Create the triggers file
-------------------------

To keep our virtualenv in sync with the host Python, let us create a ``debian/whosaysthat.triggers`` file.
The standard name for this is ``debian/packagename.triggers``.

::

    # Register interest in Python interpreter changes (Python 2 for now); and
    # don't make the Python package dependent on the virtualenv package
    # processing (noawait)
    interest-noawait /usr/bin/python3.5

    # Also provide a symbolic trigger for all dh-virtualenv packages
    interest dh-virtualenv-interpreter-update


Update the changelog file
--------------------------

First, we will copy the existing changelog file. Then, we will use ``dch`` tool to update
the entry there.

::

    $ cp ../whosaysthat-0.0.1/debian/changelog debian/
    $ dch

This will open up your favorite editor, update and save the file.


.. note:: You will have to install `devscripts` package in Debian for the `dch` command.

Create the install file
-----------------------

This is same as in the last time. Add the following in the ``debian/whosaysthat.install`` file.

::

    data/1.txt usr/share/whosaysthat/data/1.txt
    data/2.txt usr/share/whosaysthat/data/2.txt


Create a links file
--------------------

*dh-virtualenv* tool will create a virtualenv under ``/opt/venvs``, in our
example, this will be ``/opt/venvs/whosaysthat`` directory, and the console
entry point based executables will be installed in the bin directory there. So,
we should create links to those commands from ``/usr/bin``.

Add the following in the ``debian/whosaysthat.links`` file.

::

    opt/venvs/whosaysthat/bin/whatismyip usr/bin/whatismyip
    opt/venvs/whosaysthat/bin/whoisthebest usr/bin/whoisthebest



Export environment variables to use the local wheels
-----------------------------------------------------

::

    $ export DH_PIP_EXTRA_ARGS="--require-hashes --no-index --find-links=./localwheels"

This will make *dh-virtualenv* to use our wheels instead of downloading them from PyPI.


The final rules file
--------------------

Add the following text to the ``debian/rules`` file.

::

    #!/usr/bin/make -f

    %:
            dh $@ --with python-virtualenv --python /usr/bin/python3.5 --setuptools

.. note:: If you copy paste the above example, then remember to use a TAB instead of 8 spaces :)


Remember, for a package with dependent system `site-packages`, means packages which depends on
Python modules from Debian world, the above will need modification.

::

    #!/usr/bin/make -f

    %:
        dh $@ --with python-virtualenv

    override_dh_virtualenv:
        dh_virtualenv --python /usr/bin/python3.5 --setuptools -S



Let us build the package
-------------------------

::

    $ dpkg-buildpackage -us -uc

This should create the Debian package in the parent directory.
