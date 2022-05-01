.. _manual-main:

.. image:: https://img.shields.io/github/stars/dyninst/dyninst?style=social
    :alt: GitHub stars
    :target: https://github.com/dyninst/dyninst/stargazers

=======
Dyninst
=======

.. epigraph::

   *Part of the* `Paradyn <http://www.paradyn.org>`_ *project*


Dyninst is a collection of libraries for performing binary instrumentation, analysis, and
modification. These libraries are assembled into a collection of toolkits that allow users
to more effectively use different aspects of binary analysis for building their own tools.

:ref:`sec:dataflow-intro`
   Trace the flow of data through a binary using the techniques of slicing, stack analysis,
   symbolic expansion and evaluation, and register liveness

:ref:`sec:instruction-intro`
   Decode raw binary instructions into a platform-independent represention that provides
   a description of their semantics 

:ref:`sec:parseapi-intro`
   Converts the machine code representation of a program, library, or code snippet into
   platform-independent abstractions such as instructions, basic blocks, functions, and loops

:ref:`sec-patchapi-intro`
   Instrument (insert code into) and modify a binary executable or library by manipulating
   the binary’s control flow graph (CFG)

:ref:`sec:stackwalk-intro`
   Collect and analyze stack traces

:ref:`sec:symtab-intro`
   A platform-independent representation of symbol tables, object file headers, and debug information


.. _main-support:

-------
Support
-------

* For **bugs and feature requests**, please use the `issue tracker <https://github.com/dyninst/dyninst/issues>`_.
* For **contributions**, visit us on `Github <https://github.com/dyninst/dyninst>`_.

---------
Resources
---------

`GitHub Repository <https://github.com/dyninst/dyninst>`_
    The code on GitHub.

------------
Developed by
------------

|     Paradyn Parallel Performance Tools

|     Computer Science Department
|     University of Wisconsin-Madison
|     Madison, WI 53706

|     Computer Science Department
|     University of Maryland
|     College Park, MD 20742


.. toctree::
   :caption: getting started
   :name: basics
   :hidden:
   :maxdepth: 1

   overview
   building

.. toctree::
   :caption: toolkits 
   :name: toolkits
   :hidden:
   :maxdepth: 2

   dataflowAPI/index
   dyninstAPI/index
   instructionAPI/index
   parseAPI/index
   patchAPI/index
   proccontrolAPI/index
   stackwalk/index
   symtabAPI/index

.. toctree::
   :caption: user tools
   :name: usertools
   :hidden:
   :maxdepth: 3
   
   usertools/DynC/index

.. toctree::
   :caption: developer apis
   :name: dev-apis
   :hidden:
   :maxdepth: 2
   
   dataflowAPI/API
   dyninstAPI/API
   instructionAPI/API
   parseAPI/API
   patchAPI/API
   proccontrolAPI/API
   stackwalk/API
   symtabAPI/API