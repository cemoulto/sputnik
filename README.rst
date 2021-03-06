Sputnik: a data package manager library
***************************************

Sputnik is a library for managing data packages for another library, e.g., models for a machine learning library. It also comes with a command-line interface, run ``sputnik --help`` or ``python -m sputnik --help`` for assistance. Sputnik is a pure Python library licensed under MIT, has minimal dependencies (only ``semver``) and is compatible with python >=2.6 and >=3.3 on Linux, OSX and Windows.

.. image:: https://travis-ci.org/spacy-io/sputnik.svg?branch=master
    :target: https://travis-ci.org/spacy-io/sputnik

Installation
============

Sputnik is available from `PyPI <https://pypi.python.org/pypi/sputnik>`_ via ``pip``:

.. code:: python

 pip install sputnik

and from spaCy's `Anaconda <https://anaconda.org/spacy/sputnik>`_ channel via ``conda``

.. code:: python

 conda install -c https://conda.anaconda.org/spacy sputnik

Build a package
===============

Add a ``package.json`` file with following JSON to a directory ``sample`` and add some files in ``sample/data`` that you would like to have packaged, e.g., ``sample/data/model``. See a sample layout `here <https://github.com/spacy-io/sputnik/tree/master/sample>`_.

.. code:: python

 {
   "name": "my_model",
   "include": [["data", "*"]],
   "version": "1.0.0"
 }

Note that include's path components are lists to avoid platform compatibility issues.

Build the package with following code, it should produce a new file and output its path: ``sample/my_model-1.0.0.sputnik``.

.. code:: python

 import sputnik
 archive = sputnik.build('sample')
 print(archive.path)

Install a package
=================

Decide for a location for your installed packages, e.g., ``packages``. Then install the previously built package with following code, it should output the path of the now installed package: ``packages/my_model-1.0.0``

.. code:: python

 package = sputnik.install(<app_name>, <app_version>, 'sample/my_model-1.0.0.sputnik', data_path='packages')
 print(package.path)

Replace ``<app_name>`` and ``<app_version>`` with your app's name and version. This information is used to check for package compatibility. You can also provide ``None`` instead to disable package compatibility checks. Read more about package compatibility under the Compatibility section below.

List installed packages
=======================

This should output the package strings for all installed packages, e.g., ``['my_model-1.0.0']``:

.. code:: python

 packages = sputnik.find(<app_name>, <app_version>, data_path='packages')
 print([p.ident for p in packages])

Access package data
===================

Sputnik makes it easy to access packaged data files without dealing with filesystem paths or archive file formats.

First, get a Sputnik package object with:

.. code:: python

 package = sputnik.package(<app_name>, <app_version>, 'my_model', data_path='packages')

On the package object you can check for the existence of a file or directory, get it's path or directly open it. Note that each directory in a path must be provided as separate argument. Do not address paths with slashes or backslashes as this will lead to platform-compatibility issues.

.. code:: python

 if package.has_path('data', 'model'):
   with io.open(package.file_path('data', 'model'), mode='r', encoding='utf8') as f:
     res = f.read()

Alternatively you can use Sputnik's ``open()`` wrapper:

.. code:: python

 with package.open(['data', 'model'], mode='r', encoding='utf8') as f:
   res = f.read()

Note that ``package.file_path()`` only works on files, not directory. Use ``package.dir_path()`` on directories.

If you want to list all file contents of a package use ``sputnik.files('my_model', data_path='packages')``.

Remove package
==============

.. code:: python

 sputnik.remove(<app_name>, <app_version>, 'my_model', data_path='packages')

Purge package pool/cache
========================

.. code:: python

 sputnik.purge(<app_name>, <app_version>, data_path='packages')

Versioning
==========

``install``, ``find``, ``package``, ``files``, ``search`` and ``remove`` commands accept version constraint strings that follow `semantic versioning <http://semver.org/>`_, e.g.:

.. code:: python

 sputnik.install(<app_name>, <app_version>, 'my_model ==1.0.0', data_path='packages')
 sputnik.find(<app_name>, <app_version>, 'my_model >1.0.0', data_path='packages')
 sputnik.package(<app_name>, <app_version>, 'my_model >=1.0.0', data_path='packages')
 sputnik.search(<app_name>, <app_version>, 'my_model <1.0.0', data_path='packages')
 sputnik.files(<app_name>, <app_version>, 'my_model <=1.0.0', data_path='packages')
 sputnik.remove(<app_name>, <app_version>, 'my_model ==1.0.0', data_path='packages')

Multiple version constraints can be concatenated with commas, e.g., ``my_model >=1.0.0,<2.0.0``. The constraint expression is satisfied if all individual constraints are satisfied.

Compatibility
=============

Sputnik allows to specify compatibility of a package with an app's name to let an index server provide app-specific views on installable packages. An app in this context is the project that imports Sputnik (e.g., ``my_library``).

my_model/package.json:
----------------------

.. code:: python

 {
   "name": "my_model",
   "description": "this model is awesome",
   "include": ["data/*"],
   "version": "2.0.0",
   "license": "public domain",
   "compatibility": {
     "my_library": null
   }
 }

Currently no compatibility checks are performed within Sputnik code.
