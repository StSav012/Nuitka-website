.. post:: 2015/04/07 17:55
   :tags: compiler, Python, Nuitka
   :author: Kay Hayen

#######################
 Nuitka Release 0.5.12
#######################

This is to inform you about the new stable release of `Nuitka
<https://nuitka.net>`_. It is the extremely compatible Python compiler,
`"download now" </doc/download.html>`_.

This release contains massive amounts of corrections for long standing
issues in the import recursion mechanism, as well as for standalone
issues now visible after the ``__file__`` and ``__path__`` values have
changed to become run time dependent values.

***********
 Bug Fixes
***********

-  Fix, the ``__path__`` attribute for packages was still the original
   filename's directory, even in file reference mode was ``runtime``.

-  The use of ``runtime`` as default file reference mode for
   executables, even if not in standalone mode, was making acceleration
   harder than necessary. Changed to ``original`` for that case. Fixed
   in 0.5.11.1 already.

-  The constant value for the smallest ``int`` that is not yet a
   ``long`` is created using ``1`` due to C compiler limitations, but
   ``1`` was not yet initialized properly, if this was a global
   constant, i.e. used in multiple modules. Fixed in 0.5.11.2 already.

-  Standalone: Recent fixes around ``__path__`` revealed issues with
   PyWin32, where modules from ``win32com.shell`` were not properly
   recursed to. Fixed in 0.5.11.2 already.

-  The importing of modules with the same name as a built-in module
   inside a package falsely assumed these were the built-ins which need
   not exist, and then didn't recurse into them. This affected
   standalone mode the most, as the module was then missing entirely.

   .. code:: python

      # Inside "x.y" module:
      import x.y.exceptions

-  Similarly, the importing of modules with the same name as standard
   library modules could go wrong.

   .. code:: python

      # Inside "x.y" module:
      import x.y.types

-  Importing modules on Windows and macOS was not properly checking the
   checking the case, making it associate wrong modules from files with
   mismatching case.

-  Standalone: Importing with ``from __future__ import absolute_import``
   would prefer relative imports still.

-  Python3: Code generation for ``try``/``return expr``/``finally``
   could loose exceptions when ``expr`` raised an exception, leading to
   a ``RuntimeError`` for ``NULL`` return value. The real exception was
   lost.

-  Lambda expressions that were directly called with star arguments
   caused the compiler to crash.

   .. code:: python

      (lambda *args: args)(*args)  # was crashing Nuitka

**************
 Optimization
**************

-  Focusing on compile time memory usage, cyclic dependencies of trace
   merges that prevented them from being released, even when replaced
   were removed.

-  More memory efficient updating of global SSA traces, reducing memory
   usage during optimization by ca. 50%.

-  Code paths that cannot and therefore must not happen are now more
   clearly indicated to the backend compiler, allowing for slightly
   better code to be generated by it, as it can tell that certain code
   flows need not be merged.

**************
 New Features
**************

-  Standalone: On systems, where ``.pth`` files inject Python packages
   at launch, these are now detected, and taking into account.
   Previously Nuitka did not recognize them, due to lack of
   ``__init__.py`` files. These are mostly pip installations of e.g.
   ``zope.interface``.

-  Added option ``--explain-imports`` to debug the import resolution
   code of Nuitka.

-  Added options ``--show-memory`` to display the amount of memory used
   in total and how it's spread across the different node types during
   compilation.

-  The option ``--trace-execution`` now also covers early program
   initialisation before any Python code runs, to ease finding bugs in
   this domain as well.

****************
 Organisational
****************

-  Changed default for file reference mode to ``original`` unless
   standalone or module mode are used. For mere acceleration, breaking
   the reading of data files from ``__file__`` is useless.

-  Added check that the in-line copy of scons is not run with Python3,
   which is not supported. Nuitka works fine with Python3, but a Python2
   is required to execute scons.

-  Discover more kinds of Python2 installations on Linux/macOS
   installations.

-  Added instructions for macOS to the download page.

**********
 Cleanups
**********

-  Moved ``oset`` and ``odict`` modules which provide ordered sets and
   dictionaries into a new package ``nuitka.container`` to clean up the
   top level scope.

-  Moved ``SyntaxErrors`` to ``nuitka.tree`` package, where it is used
   to format error messages.

-  Moved ``nuitka.Utils`` package to ``nuitka.utils.Utils`` creating a
   whole package for utils, so as to better structure them for their
   purpose.

*********
 Summary
*********

This release is a major maintenance release. Support for namespace
modules injected by ``*.pth`` is a major step for new compatibility. The
import logic improvements expand the ability of standalone mode widely.
Many more use cases will now work out of the box, and less errors will
be found on case insensitive systems.

There is aside of memory issues, no new optimization though as many of
these improvements could not be delivered as hotfixes (too invasive code
changes), and should be out to the users as a stable release. Real
optimization changes have been postponed to be next release.
