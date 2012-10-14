Testing/development buildouts for Plone
=======================================

This contains a number of configuration files for using [zc.buildout](http://pypi.python.org/pypi/zc.buildout/) to quickly set up a testing/development environment for your package.  The intended usage it to create a ``buildout.cfg`` like::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/test-4.2.x.cfg
    package-name = plone.app.foo

Running buildout should give you a ``bin/test`` script, which can be used to run your package's tests.  If your tests have additional dependencies, you should declare them via the ``extras_require`` parameter of ``setuptools.setup`` and refer to the key used using the ``package-extras`` variable::

    [buildout]
    extends = https://raw.github.com/collective/buildout.plonetest/master/test-4.2.x.cfg
    package-name = plone.app.foo
    package-extras = [test]
