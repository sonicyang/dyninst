.. _manual-main:

=======
Dyninst
=======

.. image:: https://img.shields.io/github/stars/dyninst/dyninst?style=social
    :alt: GitHub stars
    :target: https://github.com/dyninst/dyninst/stargazers


Dyninst uses a technique called dynamic instrumentation to efficiently obtain performance profiles of unmodified executables. 
This dynamic binary instrumentation technology is independently available to researchers via the Dyninst API,
documented here.

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


.. toctree::
   :caption: toolkits 
   :name: toolkits
   :hidden:
   :maxdepth: 2

   dataflowAPI/index
   instructionAPI/index
   parseAPI/index
   patchAPI/index
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
   instructionAPI/API
   parseAPI/API
   patchAPI/API
   stackwalk/API
   symtabAPI/API