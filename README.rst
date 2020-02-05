Testing/development buildouts for Plone
=======================================

.. contents:: Content
   :depth: 2

This repository contains a number of configuration files for using
`zc.buildout`_ to quickly set up a testing/development environment for your
package.

A good example of how to use this, is `bobtemplates.plone <https://github.com/plone/bobtemplates.plone>`_.
Follow the instructions there to create a Plone add-on, and look at the created files,
especially the various buildout configs and `.travis.yml <https://github.com/plone/bobtemplates.plone/blob/master/bobtemplates/plone/addon/.travis.yml.bob>`_.

The intended usage is to create a ``buildout.cfg`` like::

    [buildout]
    extends = https://raw.githubusercontent.com/collective/buildout.plonetest/master/test-5.x.cfg
    package-name = plone.app.foo

Create a virtualenv and run buildout (mind the `-N`)::

    python -m venv
    bin/buildout -N

This should give you a ``bin/test`` script, which can be used to
run your package's tests.  If your tests have additional dependencies, you
should declare them via the ``extras_require`` parameter of
``setuptools.setup`` and refer to the key used using the ``package-extras``
variable::

    [buildout]
    extends = https://raw.githubusercontent.com/collective/buildout.plonetest/master/test-5.x.cfg
    package-name = plone.app.foo
    package-extras = [test]


Testing in Travis CI
^^^^^^^^^^^^^^^^^^^^

If you want to do `continuous integration`_ with `Travis CI`_ you can use this same buildout config.
And a ``.travis.yml`` like::

    version: ~> 1.0
    import: collective/buildout.plonetest:travis/default.yml
    python: "2.7"
    env: PLONE_VERSION=5.2

.. ATTENTION::
   This repository also has ``travis-*.cfg`` files, that try to be faster by downloading the Plone universal installer.
   This is no longer recommended, because Travis meanwhile has better caching support.
   Only the Travis config files for Plone 5.1 and earlier have been kept, to avoid breaking add-ons.
   In your own buildout config, replace ``travis-*.cfg`` with ``test-*.cfg`` and you should be fine.
   Also take over the ``cache`` setting from the ``.travis.yml`` file above.


i18ndude helper script to update po files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Include the following in your buildout configuration to create an `i18ndude`_
helper script to update the po files of your product::

    [buildout]
    extends =
        https://raw.githubusercontent.com/collective/buildout.plonetest/master/test-5.x.cfg
        https://raw.githubusercontent.com/collective/buildout.plonetest/master/qa.cfg
    package-name = plone.app.foo
    package-extras = [test]
    parts+=
        i18ndude
        rebuild_i18n-sh

After running ``bin/buildout -N``
you will find a ``bin/rebuild_i18n.sh``;
run the script
and the po files will be updated.

Domain name is taken from the ``${buildout:package-name}`` variable.
The plone domain is also included.

.. ATTENTION::
   Originally, the ``rebuild_i18n-sh`` part used a `filesystem template <https://github.com/collective/buildout.plonetest/blob/master/templates/rebuild_i18n.sh.in>`_.
   This is deprecated and no longer updated.


Functional tests with Robot Framework and SeleniumLibrary in Travis CI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Update your ``.travis.yml`` file with the following::

    before_script:
      - export DISPLAY=:99.0
      - sh -e /etc/init.d/xvfb start


Testing in Travis CI with multiple versions of Plone and Python
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``buildout.cfg`` file in your package should look like this::

    [buildout]
    extends =
        https://raw.githubusercontent.com/collective/buildout.plonetest/master/test-5.x.cfg

    package-name = collective.foo
    package-extras = [test]

    [versions]
    setuptools = 41.2.0
    zc.buildout = 2.13.2

The versions are optional, but they can help in avoiding restarts when buildout tries to upgrade itself to a version pinned in Plone.
It is fine to start without them, but when you run into problems in `Travis CI`_, consider adding them.
In this way, all Plone versions use the same buildout and setuptools versions.

These versions match a ``requirements.txt`` like this::

    setuptools==41.2.0
    zc.buildout==2.13.2

The ``.travis.yml`` file should look like this::

    version: ~> 1.0
    import: collective/buildout.plonetest:travis/default.yml
    matrix:
      include:
        - python: "2.7"
          env: PLONE_VERSION="4.3"
        - python: "2.7"
          env: PLONE_VERSION="5.1"
        - python: "2.7"
          env: PLONE_VERSION="5.2"
        - python: "3.7"
          env: PLONE_VERSION="5.2"

The trick here is to replace the extended configuration with the right one
using the `sed`_ command.


Experimental configurations
===========================

.. Caution::
    The following configurations are experimental and may change at any time.


Quality Assurance
^^^^^^^^^^^^^^^^^

If you want to add Quality Assurance to your continuous integration you can
update your ``buildout.cfg`` file like::

    [buildout]
    extends =
        https://raw.githubusercontent.com/collective/buildout.plonetest/master/test-5.x.cfg
        https://raw.githubusercontent.com/collective/buildout.plonetest/master/qa.cfg
    package-name = plone.app.foo
    package-extras = [test]
    package-min-coverage = 80
    parts+=
        createcoverage
        coverage-sh
        code-analysis

and add and commit ``.coveragerc`` file
(see example at https://github.com/plone/plone.recipe.codeanalysis/blob/master/.coveragerc)

and update your ``.travis.yml`` like::

    language: python
    python: 2.7
    cache:
      pip: true
      directories:
        - eggs
    env:
      - TARGET=test
      - TARGET=coverage.sh
    before_install:
      - virtualenv -p `which python` .
      - bin/pip install -r requirements.txt
      - bin/buildout -N -t 3 annotate
    install:
      - bin/buildout -N -t 3
    script:
      - bin/$TARGET

.. _`continuous integration`: https://en.wikipedia.org/wiki/Continuous_integration
.. _`i18ndude`: http://pypi.python.org/pypi/i18ndude/
.. _`plone.recipe.codeanalysis`: http://pypi.python.org/pypi/plone.recipe.codeanalysis/
.. _`sed`: http://www.thegeekstuff.com/2009/11/unix-sed-tutorial-append-insert-replace-and-count-file-lines/
.. _`Travis CI`: http://travis-ci.org/
.. _`zc.buildout`: http://pypi.python.org/pypi/zc.buildout/
