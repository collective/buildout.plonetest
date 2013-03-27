Testing/development buildouts for Plone
=======================================

This contains a number of configuration files for using `zc.buildout`_ to
quickly set up a testing/development environment for your package.  The
intended usage is to create a ``buildout.cfg`` like::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/test-4.2.x.cfg
    package-name = plone.app.foo

Running buildout should give you a ``bin/test`` script, which can be used to
run your package's tests.  If your tests have additional dependencies, you
should declare them via the ``extras_require`` parameter of
``setuptools.setup`` and refer to the key used using the ``package-extras``
variable::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/test-4.2.x.cfg
    package-name = plone.app.foo
    package-extras = [test]

If you want to do `continuous integration`_ with `Travis CI`_ you should
create a ``travis.cfg`` like::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/travis-4.x.cfg
    package-name = plone.app.foo
    package-extras = [test]

And a ``.travis.yml`` like::

    language: python
    python: 2.7
    install:
      - mkdir -p buildout-cache/downloads
      - python bootstrap.py -c travis.cfg
      - bin/buildout -N -t 3 -c travis.cfg
    script: bin/test

Functional tests with Robot Framework and SeleniumLibrary in Travis CI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Update your ``.travis.yml`` file with the following::

    before_script:
      - export DISPLAY=:99.0
      - sh -e /etc/init.d/xvfb start

Testing in Travis CI with multiple versions of Plone
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``travis.cfg`` file in your package should look like this::

    [buildout]
    extends =
        https://raw.github.com/collective/buildout.plonetest/master/travis-4.x.cfg

    package-name = collective.foo
    package-extras = [test]

    allow-hosts +=
        code.google.com
        robotframework.googlecode.com

And the ``.travis.yml`` file should look like this::

    language: python
    python: 2.7
    env:
      - PLONE_VERSION=4.2
      - PLONE_VERSION=4.3
    install:
      - sed -ie "s#travis-4.x.cfg#travis-$PLONE_VERSION.x.cfg#" travis.cfg
      - mkdir -p buildout-cache/downloads
      - python bootstrap.py -c travis.cfg
      - bin/buildout -c travis.cfg -N -q -t 3
    script: bin/test

The trick here is to replace the extended configuration with the right one
using the `sed`_ command.

Experimental configurations
---------------------------

.. Caution::
    The following configurations are experimental and may change at any time.

Quality Assurance
^^^^^^^^^^^^^^^^^

.. Info::
    Work on this configuration is being done right now in `another repo`_.

If you want to add Quality Assurance to your continuous integration you can
update your ``travis.cfg`` file like::

    [buildout]
    extends =
        https://raw.github.com/collective/buildout.plonetest/master/travis-4.x.cfg
        https://raw.github.com/collective/buildout.plonetest/master/qa.cfg
    package-name = plone.app.foo
    package-extras = [test]
    package-src = src/plone/app/foo
    package-pep8-ignores = E501,W402,W801
    package-coverage = 80
    parts+=
        createzopecoverage
        coverage-sh
        flake8
        python-validation-sh
        css-validation-sh
        js-validation-sh

and update your ``.travis.yml`` like::

    language: python
    python: 2.7
    env:
      - TARGET=test
      - TARGET=coverage.sh
      - TARGET=python-validation.sh
    
    #  - TARGET=css-validation.sh
    #  - TARGET=js-validation.sh
    
    # csslint and jshint dependency, uncomment if needed
    # before_install:
    #  - sudo apt-get install ack-grep
    #
    # csslint
    #  - npm install csslint -g
    #
    # jshint
    #  - npm install jshint -g
    
    install: 
      - mkdir -p buildout-cache/downloads
      - python bootstrap.py -c travis.cfg
      - bin/buildout -c travis.cfg -N -q -t 3
    
    script: bin/$TARGET

.. _`zc.buildout`: http://pypi.python.org/pypi/zc.buildout/
.. _`continuous integration`: https://en.wikipedia.org/wiki/Continuous_integration
.. _`Travis CI`: http://travis-ci.org/
.. _`sed`: http://linux.about.com/od/commands/l/blcmdl1_sed.htm
.. _`another repo`: https://github.com/hvelarde/qa
