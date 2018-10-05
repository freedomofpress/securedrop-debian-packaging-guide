Debian packaging 101
=====================

In this chapter we will package our example project into a Debian package and
learn about the process and different tools and files involved.


Install the required package build tools
-----------------------------------------

We will need the required build tools in the system.

::

    $ sudo apt-get install build-essential devscripts dh-python python3-all python3-setuptools dh-make


Get the source tarball
-----------------------

Clone the example project anywhere you like, and create an release tarball using the
following command.

::

    $ git clone https://github.com/kushaldas/whosaysthat.git
    Cloning into 'whosaysthat'...
    remote: Counting objects: 49, done.
    remote: Compressing objects: 100% (28/28), done.
    remote: Total 49 (delta 15), reused 44 (delta 11), pack-reused 0
    Unpacking objects: 100% (49/49), done.

    $ cd whosaysthat
    $ $ python3 setup.py sdist
    /usr/lib/python3.5/distutils/dist.py:261: UserWarning: Unknown distribution option: 'long_description_content_type'
    warnings.warn(msg)
    running sdist
    running egg_info
    creating whosaysthat.egg-info
    writing whosaysthat.egg-info/PKG-INFO
    writing dependency_links to whosaysthat.egg-info/dependency_links.txt
    writing entry points to whosaysthat.egg-info/entry_points.txt
    writing top-level names to whosaysthat.egg-info/top_level.txt
    writing requirements to whosaysthat.egg-info/requires.txt
    writing manifest file 'whosaysthat.egg-info/SOURCES.txt'
    reading manifest file 'whosaysthat.egg-info/SOURCES.txt'
    reading manifest template 'MANIFEST.in'
    writing manifest file 'whosaysthat.egg-info/SOURCES.txt'
    warning: sdist: standard file not found: should have one of README, README.rst, README.txt

    running check
    creating whosaysthat-0.0.1
    creating whosaysthat-0.0.1/configs
    creating whosaysthat-0.0.1/data
    creating whosaysthat-0.0.1/mike
    creating whosaysthat-0.0.1/whosaysthat
    creating whosaysthat-0.0.1/whosaysthat.egg-info
    copying files to whosaysthat-0.0.1...
    copying LICENSE -> whosaysthat-0.0.1
    copying MANIFEST.in -> whosaysthat-0.0.1
    copying README.md -> whosaysthat-0.0.1
    copying setup.py -> whosaysthat-0.0.1
    copying whatismyip -> whosaysthat-0.0.1
    copying configs/whosaysthat.json -> whosaysthat-0.0.1/configs
    copying data/1.txt -> whosaysthat-0.0.1/data
    copying data/2.txt -> whosaysthat-0.0.1/data
    copying mike/__init__.py -> whosaysthat-0.0.1/mike
    copying mike/__main__.py -> whosaysthat-0.0.1/mike
    copying whosaysthat/__init__.py -> whosaysthat-0.0.1/whosaysthat
    copying whosaysthat/__main__.py -> whosaysthat-0.0.1/whosaysthat
    copying whosaysthat.egg-info/PKG-INFO -> whosaysthat-0.0.1/whosaysthat.egg-info
    copying whosaysthat.egg-info/SOURCES.txt -> whosaysthat-0.0.1/whosaysthat.egg-info
    copying whosaysthat.egg-info/dependency_links.txt -> whosaysthat-0.0.1/whosaysthat.egg-info
    copying whosaysthat.egg-info/entry_points.txt -> whosaysthat-0.0.1/whosaysthat.egg-info
    copying whosaysthat.egg-info/requires.txt -> whosaysthat-0.0.1/whosaysthat.egg-info
    copying whosaysthat.egg-info/top_level.txt -> whosaysthat-0.0.1/whosaysthat.egg-info
    Writing whosaysthat-0.0.1/setup.cfg
    creating dist
    Creating tar archive
    removing 'whosaysthat-0.0.1' (and everything under it)
    $ ls dist/
    whosaysthat-0.0.1.tar.gz

As you can see, the final output of the above command is a tarball inside of the
``dist/`` directory.


Working directory for the packaging
------------------------------------

We will use the the ``~/packaging/`` as the working directory to build all the
packages. Create this directory in your system.

.. warning:: Do not build any package as root user.


Install the build dependencies of the package itself
-----------------------------------------------------

In this step we should install the build dependency of our package itself. As we
use ``requests`` module in the example project, we will just install that from
Debian repository.

::

    $ sudo apt-get install python3-requests


Extracting our source tarball
-----------------------------

::

    $ cp dist/whosaysthat-0.0.1.tar.gz ~/packaging
    $ cd ~/packaging
    $ $ tar -xvf whosaysthat-0.0.1.tar.gz
    whosaysthat-0.0.1/
    whosaysthat-0.0.1/setup.py
    whosaysthat-0.0.1/configs/
    whosaysthat-0.0.1/configs/whosaysthat.json
    whosaysthat-0.0.1/PKG-INFO
    whosaysthat-0.0.1/mike/
    whosaysthat-0.0.1/mike/__init__.py
    whosaysthat-0.0.1/mike/__main__.py
    whosaysthat-0.0.1/LICENSE
    whosaysthat-0.0.1/whosaysthat.egg-info/
    whosaysthat-0.0.1/whosaysthat.egg-info/PKG-INFO
    whosaysthat-0.0.1/whosaysthat.egg-info/top_level.txt
    whosaysthat-0.0.1/whosaysthat.egg-info/requires.txt
    whosaysthat-0.0.1/whosaysthat.egg-info/entry_points.txt
    whosaysthat-0.0.1/whosaysthat.egg-info/SOURCES.txt
    whosaysthat-0.0.1/whosaysthat.egg-info/dependency_links.txt
    whosaysthat-0.0.1/data/
    whosaysthat-0.0.1/data/2.txt
    whosaysthat-0.0.1/data/1.txt
    whosaysthat-0.0.1/whosaysthat/
    whosaysthat-0.0.1/whosaysthat/__init__.py
    whosaysthat-0.0.1/whosaysthat/__main__.py
    whosaysthat-0.0.1/README.md
    whosaysthat-0.0.1/MANIFEST.in
    whosaysthat-0.0.1/setup.cfg
    whosaysthat-0.0.1/whatismyip
    $ cd whosaysthat-0.0.1/

In the above commands, we extracted the tarball and cd into the source directory.

Add the packager details in your ~/.bashrc
--------------------------------------------

Edit and add the following lines to reflect the right name and email address and add it to your
``~/.bashrc`` file. Remember to source the file.

::

    DEBEMAIL="kushal@freedom.press"
    DEBFULLNAME="Kushal Das"
    export DEBEMAIL DEBFULLNAME


Create the initial packaging file
-----------------------------------

::

    $ dh_make -f ../whosaysthat-0.0.1.tar.gz
    Type of package: (single, indep, library, python)
    [s/i/l/p]?
    Email-Address       : kushal@freedom.press
    License             : blank
    Package Name        : whosaysthat
    Maintainer Name     : Kushal Das
    Version             : 0.0.1
    Package Type        : python
    Date                : Mon, 17 Sep 2018 19:51:02 -0400
    Are the details correct? [Y/n/q]
    Please respond with "yes" or "no" (or "y" or "n")
    pth
    Done. Please edit the files in the debian/ subdirectory now.


.. note:: remember that you will have to do this only for building the package for the first time.

After this we will have a new ``debian`` directory inside of the current
directory. This directory has a lot of new files required for the packaging
work.

::

    $ tree debian/
    debian/
    ├── changelog
    ├── compat
    ├── control
    ├── copyright
    ├── manpage.1.ex
    ├── manpage.sgml.ex
    ├── manpage.xml.ex
    ├── menu.ex
    ├── postinst.ex
    ├── postrm.ex
    ├── preinst.ex
    ├── prerm.ex
    ├── README.Debian
    ├── README.source
    ├── rules
    ├── source
    │   ├── format
    │   └── options
    ├── watch.ex
    ├── whosaysthat.cron.d.ex
    ├── whosaysthat.default.ex
    ├── whosaysthat.doc-base.EX
    └── whosaysthat-docs.docs

    1 directory, 22 files


Editing the control file
-------------------------

Our first step is to edit the ``control`` file and update it with the required information.

::

    Source: whosaysthat
    Section: unknown
    Priority: optional
    Maintainer: Kushal Das <kushal@freedom.press>
    Build-Depends: debhelper (>= 9), dh-python, python3-all, python3-setuptools
    Standards-Version: 3.9.8
    Homepage: https://github.com/freedomofpress/yourpackage
    X-Python-Version: >= 2.6
    X-Python3-Version: >= 3.5

    Package: whosaysthat
    Architecture: all
    Depends: ${python3:Depends}, python3-requests, ${misc:Depends}
    Description: This is our example tool
    This package installs the library for Python 3.


Import points to remember for this file.

- Double check the Build-Depends lines
- Add all the Debian packages this package is depending on the ``Depends`` line


Editing the copyright file
---------------------------

The ``debian/copyright`` is an important file which tracks the copyright details
of the different files inside of the package.


Editing the changelog
----------------------

This is a *must have* file for the package. Below is an example. #1234 is the
release ticket in our project's github.

::

    whosaysthat (0.0.1-1) unstable; urgency=medium

    * Initial release (Closes: #1234)

    -- Kushal Das <kushal@freedom.press>  Mon, 17 Sep 2018 19:51:02 -0400


.. note:: Please update this file with new entries everytime you rebuild the package with any kind of change.


The rules file
---------------

This is primary file which decides how the package will be built. We can just simply use
the standard commands provided by our *dh* tools. For more details, please have a look
at the `documentation <https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#rules>`_.

The following should be a good start for a `setup.py` based project.

::


    #!/usr/bin/make -f
    # See debhelper(7) (uncomment to enable)
    # output every command that modifies files on the build system.
    #export DH_VERBOSE = 1

    export PYBUILD_NAME=whosaysthat

    %:
            dh $@ --with python2,python3 --buildsystem=pybuild


Copying extra files to different directories
---------------------------------------------

We should a new ``debian/packagename.install`` file for the same.
For our example package, we will only install the data files under
``/usr/share/whosaysthat`` directory.

::

    data/1.txt usr/share/whosaysthat/data/1.txt
    data/2.txt usr/share/whosaysthat/data/2.txt


Building the package
---------------------

::

    $ dpkg-buildpackage -us -uc

This command will build the package in ``~/packaging`` directory.
