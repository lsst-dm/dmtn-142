..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   We frequently do analysis in notebooks, but then the code needs to be shared in other contexts.
   While it's possible to import code from notebooks, factoring shared code into libraries is a more maintainable option.
   This technote lays out suggestions for best practices when writing code in a hybrid interactive analysis/shared library mode.

   Over the last 18 months, we have seen a rise in the usage of the notebook aspect of the LSST Science Platform.
   As more users have come to the platform, the number of notebooks has also increased.
   This has happened relatively organically with only passing thought lent to how to organize, grow, and test the ecosystem of tools developed through notebooks.

   This document is intended as a starting place for generating a set of policies and recommendations for teams using notebooks to do analysis and produce tooling.

.. Add content here.

Notebooks as a platform for development
=======================================

Notebooks provide a nice environment for exploratory and pedagogical code development.
Some reasons they are good for this type of work are:

- They incorporate tab completion and easy access to help information
- Documentation can reside with the code via markdown cells
- Cells can be executed in arbitrary order, so the entire script doesn't need to be run to iterate on a particular part of the notebook
- Code in cells can be flexible.  E.g. one cell can execute bash commands while the next cell contains python

The same reasons notebooks perform well in exploratory scenarios are the reasons they are somewhat non-optimal for rigorous software engineering.
A concrete example is that running cells in arbitrary order makes it easy to end up with a notebook in a state where it will not successfully complete when run sequentially.
It is also possilbe for a notebook to return without error when cells are run sequentially, but give erroneous results because the cells were run in a different order when developing the notebook.
This makes testing tricky.

Commissioning and science validation are, by their nature, exploratory, analytical exercises.
This makes notebooks a natural place to start.
However, we expect initial exploratory efforts to result in tooling we will want to execute over and over, including potentially in a batch system.
With these two competing desiderata, we need a plan for how to transition between the two contexts: exploration vs. concrete tooling.

A plan
======

I will not go into aspects that are part of the standard DM development workflow here.
Instead I will focus on the aspects specific to development with notebooks.

Repository organization
-----------------------
As a general rule, notebooks should be kept in a single git repository for each broad activity.
E.g. a repository for commissioning and a repository for DM Science Validation.

The repositories should have a defined structure.
Informed by the data management git repositories, I suggest the following:

- ``notebooks/wip`` -- A directory that holds notebooks that are works in progress.
  These need not adhere to any specific quality specifications, but should have enough documentation to be readable by someone else.
- ``notebooks/prod`` -- Notebooks meant to adhere to specific quality guidelines.
  Notebooks cannot be added to ``prod`` from ``WIP`` without a review.
- ``bin`` -- A place to put command line tools.
    Command line tools are constructed from python modules as much as possible to improve test coverage.
    I.e. any code logic should be factored into functions/methods in modules in the ``python`` directory.
- ``python`` -- Python library code goes here.
- ``tests`` -- A place for unit tests of the modular python code.
  
These repositories will be ``eups`` packages so that we can use all the existing infrastructure.

Notes on notebook review
------------------------

Notebooks should be reviewed by someone other than the original author.
The review should look for several things:

- The notebook should adhere to a standard templated format (not defined here).
  At the very least, the notebook should state the end product from the notebook, the responsible author, and the version of the stack it is compatible with (perhaps computed by the CI system).
- When possible, the notebook should run sequentially from top to bottom.
- The reviewer should also look at the markdown cells for accuracy.
- Any duplication of code between this and another notebook should be identified and factored out into a module.
- Reviews should attempt to promote accepted idioms for common tasks.

General suggestions
-------------------

Notebook repositories should be testable.
We do not have a notebook continuous integration system yet, but when it exists, these repositories should regularly have both unit tests and notebook tests run.
This only applies to notebooks in ``notebooks/prod``.

Be aggressive about moving common code into python modules rather than letting multiple copies bit rot in separate notebooks.

Have a concrete plan for what to do about notebooks that do not pass testing.
At some point they should be moved back to ``notebooks/WIP``, but whether that is immediate or there is a grace period for the author to fix the notebook should be up to the repository owner.

Whether notebooks are cleared before committing, will depend on the intentions of the team using the notebook repository.

- Clearing outputs will reduce the overall size, make the diff cleaner, and makes the notebook clean for the next person to execute
- Leaving outputs in the committed notebook will save time since the notebook does not need to be executed to see plots etc.
  Leaving outputs also allows for robust testing since ``nbeval`` can compare outputs from the tested notebook match those of the input notebook.
  A convention was adopted in the context of data management acceptance testing of keeping both the raw notebook and the notebook containing outputs in separate directories.
  See `DMTR-201`_ and `DMTR-161`_ as examples.

.. _DMTR-201: https://github.com/lsst-dm/DMTR-201
.. _DMTR-161: https://github.com/lsst-dm/DMTR-161

Notebooks should specify what platform they are intended for: e.g. LSP at LDF, lab, etc.
Notebooks in ``notebooks/prod`` must be executable on the LSP at the LDF since that is where continuous integration will be carried out.

Strive to use a common style in code cells.
The DM team uses the `PEP-8`_.
There are many auto-formatters out there.

.. _PEP-8: https://developer.lsst.io/python/style.html?highlight=pep#pep-8-is-the-baseline-coding-style

The general guideline is to keep module libraries in the same repository as the notebooks which use them.
However, it is obvious there will be situations where different repositories will want to share imported code.
For example, the commissioning team and science validation team will certainly have overlapping analysis tasks that could make use of shared code.
In these cases, we suggest have another repository of utility/library code that is a dependency of both the higher level notebook repositories.
This will require coordination on code review to reduce duplication.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
