Notes to the Python developer
==============================

Even before we start discussing Debian packaging, first we are going to look
into a few guideline for the project development based on Python.

All new projects will be maintained for Debian Stretch, it means all of code has
to run on ``Python 3.5.3``. This means we have to make sure we do not use any
later language feature or any module which is dependent on the latest Python
version. ``Python 3.5.3`` is our runtime dependency.

But, this does not mean that we can not use tools which are dependent on the
latest version of Python. For example, `Black
<https://black.readthedocs.io/en/stable/>`_ requires at least Python 3.6, and we
can easily have it on the developer's system (say using a container or inside of
a vm), and make sure that we format our code using Black. But, this is not a
runtime dependency.

Example project git repository
-------------------------------

We will use the `whosaysthat <https://github.com/kushaldas/whosaysthat>`_
repository as the example project for the rest of this documentation.


Check security implication before using any new Python module
--------------------------------------------------------------

Before you start using any new Python module from PyPI, make sure you get a
sign-off from the rest of the team. You should look for the number of
maintainers, and active issue-tracker and other users of any module before start
using it for a project.

Use Pipenv for development environment setup
---------------------------------------------

Use `pipenv <https://pipenv.readthedocs.io/en/latest/>`_ tool to maintain your
project's dependencies.

MANIFEST.in file
-----------------

All of the files which should be installed into the end user system, should be
mentioned in the ``MANIFEST.in`` file. `Here
<https://github.com/kushaldas/whosaysthat/blob/master/MANIFEST.in>`_ is one
example of our example project.


Writing your setup.py
----------------------

The ``setup.py`` is key point deciding how the project will get installed. If
you want to know about all the available options, `this guide
<https://packaging.python.org/guides/distributing-packages-using-setuptools/>`_
will help you to find the details.

- Follow `semver guide <https://semver.org/>`_ for the version number of the project.
- Add a correct LICENSE file in the project repository.
- All command line tools should be marked clearly as `console entry points <https://packaging.python.org/guides/distributing-packages-using-setuptools/#entry-points>`_
- Add a ``__main__.py`` as required in the package/module so that it can be invoked as ``python3 -m modulename``.
- Remember that ``pipenv install -e .`` will enable the console scripts while you are developing the application.
- Do not install or copy any configuration file or other data using setup.py, we will use Debian package for the same.

Releasing a project
-------------------

``python3 setup.py sdist`` command will create a tarball of the package under
``dist/`` directory. Please verify the STDOUT output of the above mentioned
command as it will tell you what all files where put inside of the tar file. For
any modification, update the ``MANIFEST.in`` file accordingly.

The maintainer can then GPG sign and release the tarball in a pre-defined
location.
