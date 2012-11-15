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

.. _`zc.buildout`: http://pypi.python.org/pypi/zc.buildout/
.. _`continuous integration`: https://en.wikipedia.org/wiki/Continuous_integration
.. _`Travis CI`: http://travis-ci.org/
