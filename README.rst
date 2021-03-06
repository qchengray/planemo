.. figure:: https://raw.githubusercontent.com/jmchilton/planemo/master/docs/planemo_logo.png
   :alt: Planemo Logo
   :align: center
   :figwidth: 100%
   :target: https://github.com/galaxyproject/planemo

Command-line utilities to assist in building and publishing Galaxy_ tools.

.. image:: https://readthedocs.org/projects/planemo/badge/?version=latest
   :target: http://planemo.readthedocs.io/en/latest/?badge=latest
   :alt: Documentation Status

.. image:: https://badge.fury.io/py/planemo.svg
   :target: https://pypi.python.org/pypi/planemo/
   :alt: Planemo on the Python Package Index (PyPI)

.. image:: https://travis-ci.org/galaxyproject/planemo.png?branch=master
   :target: https://travis-ci.org/galaxyproject/planemo
   :alt: Build Status

.. image:: https://coveralls.io/repos/galaxyproject/planemo/badge.svg?branch=master
   :target: https://coveralls.io/r/galaxyproject/planemo?branch=master
   :alt: Coverage Status

* Free software: Academic Free License version 3.0
* Documentation: https://planemo.readthedocs.io.
* Code: https://github.com/galaxyproject/planemo


Quick Start
-----------

This quick start demonstrates using ``planemo`` commands to help
develop Galaxy tools.

-----------------
Obtaining Planemo
-----------------

For a traditional Python installation of Planemo, first set up a virtualenv
for ``planemo`` (this example creates a new one in ``.venv``) and then
install with ``pip``. Planemo requires pip 7.0 or newer.

::

    $ virtualenv .venv; . .venv/bin/activate
    $ pip install "pip>=7" # Upgrade pip if needed.
    $ pip install planemo

Another approach for installing Planemo is to use Homebrew_ or
linuxbrew_. To install Planemo this way use the ``brew`` command as
follows.

::

    $ brew tap galaxyproject/tap
    $ brew install planemo

For information on updating Planemo, installing the latest development release,
or installing planemo via bioconda - checkout the `installation
<http://planemo.readthedocs.io/en/latest/installation.html>`__ 
documentation.

Planemo is also available as a `virtual appliance
<https://planemo.readthedocs.io/en/latest/appliance.html>`_ bundled
with a preconfigured Galaxy server and set up for Galaxy tool development.
You can choose from open virtualization format (OVA_, .ova), Docker_,
or Vagrant_ appliances.

--------------
Planemo Basics
--------------

This quick start will assume you have a directory with one or more
tool XML files. If none is available, one can be quickly created for
demonstrating ``planemo`` as follows ``mkdir mytools; cd mytools; planemo
project_init --template=demo``.

Planemo can check tools containing XML for common problems and best
practices using the ``lint`` `command <http://planemo.readthedocs.org/en/latest/commands.html#lint-command>`_
(also aliased as ``l``).

::

    $ planemo lint

Like many ``planemo`` commands - by default this will search the
current directory and use all tool files it finds. It can be explicitly
passed a path to tool files or a directory of tool files.

::

    $ planemo l randomlines.xml

The ``lint`` command takes in additional options related to
reporting levels, exit code, etc. These options are described
in the `docs <http://planemo.readthedocs.org/en/latest/commands.html#lint-command>`_
or (like with all commands) can be accessed by passing ``--help`` to it.

::

    $ planemo l --help
    Usage: planemo lint [OPTIONS] TOOL_PATH

Once tools are syntactically correct - it is time to test. The ``test``
`command <http://planemo.readthedocs.org/en/latest/commands.html#test-command>`__
can be used to test a tool or a directory of tools.

::

	$ planemo test --galaxy_root=../galaxy randomlines.xml

If no ``--galaxy_root`` is defined, ``planemo`` will check for a default in
`~/.planemo.yml
<http://planemo.readthedocs.org/en/latest/configuration.html>`_ and finally
search the tool's parent directories for a Galaxy root directory.
Planemo can also download and configure a disposable Galaxy instance for
testing. Pass ``--install_galaxy`` instead of ``--galaxy_root``.

::

	$ planemo t --install_galaxy

Planemo will create a HTML output report in the current directory named
``tool_test_output.html`` (override with ``--test_output``). See an
`example <http://galaxyproject.github.io/planemo/tool_test_viewer.html?test_data_url=https://gist.githubusercontent.com/jmchilton/9d4351c9545d34209904/raw/9ed285d3cf98e435fc4a743320363275949ad63c/index>`_
of such a report for Tophat.

Once tools have been linted and tested - the tools can be viewed in a
Galaxy interface using the ``serve`` (``s``) `command
<http://planemo.readthedocs.org/en/latest/commands.html#serve-command>`__.

::

	$ planemo serve

Like ``test``, ``serve`` requires a Galaxy root and one can be
explicitly specified with ``--galaxy_root`` or installed dynamically
with ``--install_galaxy``.

---------
Tool Shed
---------

Planemo can help you publish tools to the Galaxy Tool Shed.
Check out `Publishing to the Tool Shed`_.


Experimental Features
---------------------

Planemo can also be used to explore some more experimental features related to
Galaxy tooling - including support for Conda_, `Travis CI`_, Docker_,
`Common Workflow Language`_ (CWL), and Homebrew_.

---------------------
Conda Package Manager
---------------------

Conda_ is package manager that among many other things can be used
to manage Python packages. Please read a few notes regarding the setup:

* Planemo cannot be installed via Conda and then run Galaxy inside
  the same Conda environment currently.

* `galaxy-lib <https://github.com/galaxyproject/galaxy-lib>`_ is a
  dependency of Planemo and having this on the Python path prevents
  correct operation of Galaxy.

* Conda dependency resolution can be used for Galaxy tools,

    This allows Galaxy to map conda recipes to ``requirement`` tags on tools

    Just install planemo normally via pip into a virtual environment or via brew and
    use the appropriate commands and options:

    ::

        $ planemo conda_init
        $ planemo conda_install .
        $ planemo test --galaxy_branch release_16.04 --conda_dependency_resolution .
        $ planemo serve --galaxy_branch release_16.04 --conda_dependency_resolution .

    The test and serve commands above require the target Galaxy to be 16.01 or higher.

* Galaxy can also simply run inside an existing conda environment and
  rather than using dependency resolution Conda packages can just be picked
  up from the ``PATH``.

    Note: This isn't a well supported approach yet, and when possible Planemo
    and Galaxy should not be installed inside of Conda and the conda dependency
    resolution (as described above) should be used to allow tools to find Conda
    packages.

    To run Galaxy within a Conda environment, Planemo must be installed outside the
    Conda environment - via pip into a virtual environment or via homebrew. In the
    former case, don't place the virtualenv's ``bin`` directory on your PATH - it
    will conflict with Conda. Just reference Planemo directly
    ``/path/to/planemo_venv/bin/planemo test``.

    To run Galaxy from within the environment you will need to install Galaxy
    dependencies into the conda environment. Target the development branch of Galaxy
    for this since it has "unpinned" dependencies that are easier to fullfill for
    conda.

    ::

        (conda-env-test) $ conda install numpy bx-python pysam # install the hard ones using conda
        (conda-env-test) $ cd $GALAXY_ROOT
        (conda-env-test) $ ./scripts/common_startup.sh --skip-venv --dev-wheels # install remaining ones using pip via Galaxy
        (conda-env-test) $ cd /path/to/my/tools
        (conda-env-test) $ /path/to/planemo_venv/bin/planemo test --skip_venv  .
        (conda-env-test) $ /path/to/planemo_venv/bin/planemo serve --skip_venv  .

    A small `test script <https://github.com/galaxyproject/planemo/blob/master/tests/scripts/conda_test.sh>`__
    in the Planemo source tree demonstrates this.

-----------
TravisCI
-----------

When tools are ready to be published to GitHub_, it may be valuable to setup
continuous integration to test changes committed and new pull requests.
`Travis CI`_ is a service providing free testing and deep integration with
GitHub_.

The ``travis_init`` `command
<http://planemo.readthedocs.org/en/latest/commands.html#travis_init-
command>`__ will bootstrap a project with files to ease continuous integration
testing of tools using a Planemo driven approach inspired by this great `blog
post <http://bit.ly/gxtravisci>`_ by `Peter Cock
<https://github.com/peterjc>`_.

::

    $ planemo travis_init .
    $ # setup Ubuntu 12.04 w/tool dependencies
    $ vim .travis/setup_custom_dependencies.bash
    $ git add .travis.yml .travis
    $ git commit -m "Add Travis CI testing infrastructure for tools."
    $ git push # and register repository @ http://travis-ci.org/

In this example the file ``.travis/setup_custom_dependencies.bash`` should
install the dependencies required to test your files on to the Travis user's
``PATH``.

This testing approach may only make sense for smaller repositories with a
handful of tools. For larger repositories, such as `tools-devteam`_ or
`tools-iuc`_ simply linting tool and tool shed artifacts may be more feasible.
Check out the ``.travis.yml`` file used by the IUC as `example
<https://github.com/galaxyproject/tools-iuc/blob/master/.travis.yml>`__.

-----------------
Docker Containers
-----------------

Galaxy has `experimental support
<https://wiki.galaxyproject.org/Admin/Tools/Docker>`_ for running jobs in
Docker_ containers. Planemo contains tools to assist in development of Docker
images for Galaxy tools.

A shell can be launched to explore the Docker environment referenced by tools
that are annotated with publically registered Docker images.

::

    $ (planemo docker_shell bowtie2.xml)

For Docker containers still in development - a Dockerfile can be associated
with a tool by sticking it in the tool's directory. Planemo can then build
and tag a Docker image for this tool and launch a shell into it using the
following commands.

::

    $ planemo docker_build bowtie2.xml # assumes Dockerfile in same dir
    $ (planemo docker_shell --from_tag bowtie2.xml)

For more details see the documentation for the `docker_build
<http://planemo.readthedocs.org/en/latest/commands.html#docker_build-command>`__
and `docker_shell
<http://planemo.readthedocs.org/en/latest/commands.html#docker_shell-command>`__
commands.

--------------------------
Common Workflow Language
--------------------------

Planemo includes highly experimental support for running a subset of valid
`Common Workflow Language`_ (CWL) tools using a fork of Galaxy enhanced to run
CWL tools.

::

    $ planemo project_init --template cwl_draft3_spec
    $ planemo serve --cwl cwl_draft3_spec/cat1-tool.cwl

-----------
Brew
-----------

The Galaxy development team was exploring (the effort has since shifted towards
Conda_) different options for integrating Homebrew_ and linuxbrew_ with Galaxy.
One angle is resolving the tool requirements using ``brew``. An experimental
approach for versioning of brew recipes will be used. See full discussion on
the homebrew-science issues page
`here <https://github.com/Homebrew/homebrew-science/issues/1191>`_.
Information on the implementation can be found
https://github.com/jmchilton/platform-brew until a more permanent project home
is setup.

::

    $ planemo brew_init # install linuxbrew (only need if not already installed)
    $ planemo brew # install dependencies for all tools in directory.
    $ planemo brew bowtie2.xml # install dependencies for one tool
    $ which bowtie2
    bowtie2 not found
    $ . <(planemo brew_env --shell bowtie2.xml) # shell w/brew deps resolved
    (bowtie2)$ which bowtie2
    /home/john/.linuxbrew/Cellar/bowtie2/2.1.0/bin/bowtie2
    (bowtie2)$ exit
    $ . <(planemo brew_env bowtie2.xml) # or just source deps in cur env
    $ which bowtie2
    /home/john/.linuxbrew/Cellar/bowtie2/2.1.0/bin/bowtie2

For more information see the documentation for the `brew
<http://planemo.readthedocs.org/en/latest/commands.html#brew-command>`__
and `brew_env
<http://planemo.readthedocs.org/en/latest/commands.html#brew_env-command>`__ commands.

.. _Galaxy: http://galaxyproject.org/
.. _GitHub: https://github.com/
.. _Conda: http://conda.pydata.org/
.. _Docker: https://www.docker.com/
.. _Homebrew: http://brew.sh/
.. _linuxbrew: https://github.com/Homebrew/linuxbrew
.. _Vagrant: https://www.vagrantup.com/
.. _Travis CI: http://travis-ci.org/
.. _`tools-devteam`: https://github.com/galaxyproject/tools-devteam
.. _`tools-iuc`: https://github.com/galaxyproject/tools-iuc
.. _Publishing to the Tool Shed: http://planemo.readthedocs.org/en/latest/publishing.html
.. _Common Workflow Language: http://common-workflow-language.github.io
.. _OVA: https://en.wikipedia.org/wiki/Open_Virtualization_Format
