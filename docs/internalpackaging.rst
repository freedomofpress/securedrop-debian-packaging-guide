Making your project ready for using virtualenv in Debian package
=================================================================

For this part of the tutorial, we will add a new dependency ``cryptography`` to
our example project, and we will also import the library inside of our source
code.



Change the example code
------------------------

First, we will move into the source directory and checkout a different
branch. Ensure that ``Pipfile.lock`` exists in the current working directory
and then:

::

    cd ~/code/whosaysthat/
    git checkout fancyrelease
    pipenv lock -r > requirements.txt



Work on Debian packaging
-------------------------

Then, we will create a new source tarball for our project and also copy the wheels.

::


    python3 setup.py sdist
    realpath dist/whosaysthat-0.0.2.tar.gz
    /home/user/code/whosaysthat/dist/whosaysthat-0.0.2.tar.gz


Check out our debian packaging repo
====================================

::

    git clone https://github.com/freedomofpress/securedrop-debian-packaging


Now, we will install all required dependencies and will also syncs the wheels locally.

::

    $ make install-deps
    $ make syncwheels



Create a project directory
---------------------------

::

    mkdir -p whosaysthat/debian

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
            dh $@ --with python-virtualenv --python /usr/bin/python3.5 --setuptools --index-url https://dev-bin.ops.securedrop.org/simple

.. note:: If you copy paste the above example, then remember to use a TAB instead of 8 spaces :)


Remember, for a package with dependent system `site-packages`, means packages which depends on
Python modules from Debian world, the above will need modification.

::

    #!/usr/bin/make -f

    %:
        dh $@ --with python-virtualenv

    override_dh_virtualenv:
        dh_virtualenv --python /usr/bin/python3.5 --setuptools -S --index-url https://dev-bin.ops.securedrop.org/simple



Let us build the package
-------------------------

::

    $ PKG_PATH=/home/user/code/whosaysthat/dist/whosaysthat-0.0.2.tar.gz PKG_VERSION=0.0.2 PKG_NAME=whosaysthat ./scripts/build-debianpackage

This should create the Debian package in the ``~/debbuild/whosaysthat`` directory.


For the official projects, we already have makefile targets.

::

    PKG_PATH=/home/user/code/securedrop-client/dist/securedrop-client-0.0.1.tar.gz PKG_VERSION=0.0.1 make securedrop-client


