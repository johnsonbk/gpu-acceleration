#########
Compiling
#########

Programming environment
=======================

In order to get Nvtx working, the ``nvhpc/21.9`` module must be available and
loaded. In the ``mkmf.template`` files, we have been using an additional
variable, ``ACCFLAGS`` to set options for the ``nvfortran`` compiler.

Compiler flags
==============

To run the DART ``get_close`` kernel, these are the additional compiler flags:

.. code-block::

   ACCFLAGS = -acc -ta=tesla:cc70,deepcopy,pinned -Minfo=accel -Mnofma -r8


- ``deepcopy`` is of particular concern for us.  DART has a lot of nested
  derived types: ``type%type%type``.  The compiler was not reliably able to
  determine that the nested types needed copying to the GPU.  The deepcopy flag
  forces this, but ideally you would not force a deep copy on everything. 
  Improvements to the compiler would be needed to fix this. There is a
  workaround for forcing the correct copy in the code, which is adding a loop
  around the openACC directives. However, this is not good for code readability
  as it looks like a pointless loop.
- ``Mnofma`` was to force less optimization while debugging.
- ``r8`` was to force double precision type conversions. This was a sanity
  check while debugging memory problems.  It was not needed in the end.
- ``Minfo=accel`` prints out at compile time what the compiler was able to
  parallelize.  It is similar to the old intel ``-vec-report`` flag.
- ``cc70`` is the compute capability, so this depends on the graphics card. 
  Ascent (Oak Ridge's machine) and Casper are V100 gpus so you use ``cc70``. 
  Perlmutter is A100 (same as Derecho) so you use ``cc80``. This is not
  intuitive at all for users.

General performance results
===========================

``ACCFLAGS = -acc -ta=tesla:cc80,deepcopy`` (3x)

``ACCFLAGS = -acc -ta=tesla:cc80,deepcopy,pinned`` (15x)

