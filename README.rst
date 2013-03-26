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
create a ``buildout.cfg`` like::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/travis-4.x.cfg
    package-name = plone.app.foo
    package-extras = [test]

And a ``.travis.yml`` like::

    language: python
    python:
      - "2.7"
    install:
      - mkdir -p buildout-cache/eggs
      - mkdir -p buildout-cache/downloads
      - python bootstrap.py -c travis.cfg
      - bin/buildout -N -t 3 -c travis.cfg install download install
      - bin/buildout -N -t 3 -c travis.cfg
    script: bin/test


Other helpful configurations
----------------------------

.. Caution::
    The following configurations are experimental and may change at any time.

Quality Assurance
^^^^^^^^^^^^^^^^^

If you want to add Quality Assurance to your continuous integration you can
update your travis.cfg file like::

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

and update your travis.yml like::

    language: python
    python: 2.7
    env:
      - TARGET=test
      - TARGET=coverage.sh
      - TARGET=python-validation.sh
    
    #  - TARGET=css-validation.sh
    #  - TARGET=js-validation.sh
    
    # csslint and jshint dependencies, uncomment if needed
    # before_install:
    #  - sudo apt-get install ack-grep
    #  - sudo apt-add-repository ppa:chris-lea/node.js -y
    #  - sudo apt-get update 1>/dev/null
    #  - sudo apt-get install nodejs npm -y
    #
    # csslint
    #  - npm install csslint -g
    #
    # jshint
    #  - npm install jshint -g
    #
    # robotframework or selenium
    #  - export DISPLAY=:99.0
    #  - sh -e /etc/init.d/xvfb start
    
    install: 
      - mkdir -p buildout-cache/eggs
      - mkdir -p buildout-cache/downloads
      - python bootstrap.py -c travis.cfg
      - bin/buildout -N -t 3 -c travis.cfg install download install
      - bin/buildout -N -t 3 -c travis.cfg
    
    script: bin/$TARGET

Testing in Travis CI with multiple versions of Plone
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Extend from the travis-multiversion.cfg configuration as follows:

    [buildout]
    extends =
        https://raw.github.com/collective/buildout.plonetest/master/travis-multiversion.cfg
    package-name = plone.app.foo
    package-extras = [test]

.. _`zc.buildout`: http://pypi.python.org/pypi/zc.buildout/
.. _`continuous integration`: https://en.wikipedia.org/wiki/Continuous_integration
.. _`Travis CI`: http://travis-ci.org/
