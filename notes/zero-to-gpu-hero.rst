#############################
Zero to GPU hero with OpenACC
#############################

Overview
========

`Jeff Larkin of Nvidia <https://www.nvidia.com/en-us/on-demand/search/?q=Jeff%20Larkin>`_
produced a lecture `Zero to GPU hero with Open ACC
<https://www.nvidia.com/en-us/on-demand/session/gtcspring21-s31816/>`_.

He notes that there are basically three approaches to writing parallelized
code:

1. Parallel syntax in a standard language
2. Directives in OpenACC
3. CUDA

Standard languages
------------------

C++ and FORTRAN (2008) have GPU accelerated features in the languages already.
Here is an example from FORTRAN:

.. code-block:: fortran

   do concurrent (i = 1:n)
      y(i) = y(i) + a*x(i)
   enddo

Directives
----------

Compiler directives are a middle ground between standard facets of a given
language and CUDA. The OpenACC directives annotate your existing code to give a
specialized compiler additional information (in comments) that can allow for
parallelization.

Such compiler directives enhance the language and enable the code to run well
on GPUs when it is compiled on specific compilers, such as the Nvidia HPC
compilers.

.. code-block:: fortran

   !$acc data copy(x,y)

   ...

   do concurrent (i = 1:n)
      y(i) = y(i) + a*x(i)
   enddo

   ...

   !$acc end data


CUDA
----

Maximum level of performance but at the cost of portability. This code runs
well on GPUs but isn't available anywhere else -- e.g. it can't run on CPUs.

.. code-block::

   attribute(global)
   subroutine saxpy(n, a, x, y) {

     int i = blockId%x*blockDim%x + threadIdx%x;

     if (i < n) y(i) += a*x(i)
   }

   program main
     real         :: x(:), y(:)
     real,devince :: d_x(:), d_y(:)
     d_x = x
     d_y = y

     call saxpy
       <<<(N+255)/256,256>>>(...)

    y = d_y

Amdahl's Law
============

The performance gains of your application made available by parallelization are
constrained by what serial steps such as data movement (both I/O and to and
from the GPU) and atomic operations.

Types of directives
===================

1. Initializing parallel execution
2. Managing data movement (if the CPU and GPU have distinct memory)
3. Optimization, such as loop mapping

Directive syntax
================

.. code-block:: fortran

   !$acc directive clauses

Think of the directive like a function call and the clauses as arguments passed
to the function.

CUDA managed memory (aka CUDA unified virtual memory)
=====================================================

Fundamentally, the CPU has access to a large block of relatively slow system
memory, while the GPU has access to a smaller block of faster GPU memory.

Passing data between these two memory devices is a serial process that is
bandwidth limited. The link between the two memories is PCI-Express or NVLink.

With CUDA managed memory, the programmer does not need to decide where the data
resides. This distinction is handled by the operating system and the GPU
driver.

miniWeather
===========

This tutorial uses the `miniWeather fluid dynamics application <https://github.com/mrnorman/miniweather>`_.

General steps
=============

First step
----------

Gather the application's initial performance profile. There are many tools for
doing this, such as NVIDIA Nsight Systems, gprof, Tau, Vampir, or HPCToolkit.

Adding directives
-----------------

By adding an "acc parallel loop" directive, the compiler is made aware that the
programmer wants to parallelize the loop and that it is safe to do so.

This example is in C++:

.. code-block::

   #pragma acc parallel loop collapse(2)
   private(ll,s,inds,stencil,vals,d3_vals,r,u,w,t,p)

The ``collapse(2)`` clause tells the compiler to parallelize the first *and*
second loops in the code. **GPUs excel when there are more opportunities for
parallelization.** It's a good idea to aggressively use the collapse clause to
enable as much parallelism as possible by the compiler.

The ``private`` clause tells the compiler that each iteration in the parallel
computation needs its own copy of the private variables in order to prevent a
race condition.

Keep in mind, the ``parallel`` and ``loop`` directives are two distinct
directives. The ``parallel`` directive creates "gangs" or multiple memory
blocks, and the ``loop`` directive specifies which loops to parallelize. Most
often, however, the directives are written together as ``parallel loop``.

Data optimization
-----------------

Managed memory (unified virtual memory) suffices much of the time for
optimizing the flow of memory between the CPU and GPU. However, there are
``data`` directives for explicit data management. This is an advanced topic
that is beyond the scope of Larkin's talk.


Reduction directive
-------------------

The last directive to add is the reductions directive. It contributes very
little to performance, but it is necessary to add the entire time step to the
GPU. When every iteration of a loop is doing an operation onto a particular
variable, such as a tendency variable, the reduction clause is needed to
prevent a data race.

The reduction clause is told which operation is being performed (such as
addition) and which variables the operation is being performed upon.

.. code-block::

   #pragma acc parallel loop collapse(2) reduction(+:mass_loc, te_loc)

The compiler itself is good at detecting reductions. If the programmer doesn't
add the reduction directive, the compiler might print an ``Minfo`` message
notifying the user that it added an implicit reduction clause to a loop. When
``Minfo`` does this, go back and add the clause explicitly.

Compiling
=========

Nvidia's HPC software development toolkit, which contains compilers among other
tools, is known as NVHPC. The command to compile fortran is ``nvfortran``. The
compilers support GPU and CPUs as well, from x86, to ARM to Power.

Compiler options:

- ``-acc`` enable OpenACC support
- ``-gpu`` provides the option to use managed memory: ``-gpu=managed`` makes
  data visible on both the CPU and GPU
- ``-Minfo=accel`` prints compiler feedback on how your code was accelerated
  (very useful)

The ``Minfo`` reporting also tells you when the compiler **couldn't**
accelerate the code, giving you an opportunity to go back and fix any problems.

Sometimes performance actually decreases because of the serial movement of data
between the CPU and the GPU. Jeff Larkin says, "Don't worry, it will go away."
When we use a performance profiler, we'll see when data movement occurs, and we
can start optimizing the code to minimize data movement. Our goal is to make
the computations run on the GPU for the entire time step.

Nvidia Nsight systems
=====================

This program provides a GUI to show what exactly is happening with the GPU
kernels and the movement of data between CPU and GPU.

.. note::

   The NVTX section of the Nvidia Nsight window shows the calls to the
   function. If there are gaps between function calls, you should determine why
   the gaps exist and fill in the time with processing on the GPU.

Look at which function calls are preceded and succeeded by gaps and examine
the source code. If there is another function that does computation on the CPU
then the code might be triggering page faults. Ensure that the data stays on
the GPU and doesn't get moved back and forth between the CPU and the GPU.

In the case of ``compute_tendencies_z`` there is a loop after the function call
that adds the tendencies to the fluid state:

.. code-block::

   else if (dir == DIR_Z) {
     compute_tendencies_z(state_forcing, flux, tend);
   }

   ...

   state_out[inds] = state_init[inds} + dt * tend[indt];

This reference to ``tend[indt]`` is causing the page fault.

Final profiling
---------------

If done correctly, the GPU acceleration will dramatically speed up serial code.
It no longer makes sense to compare the serial code to the GPU-enhanced code.
Instead, the full-socket MPI performance should be compared against the
GPU-enhanced performance.

If an application is threaded, then the threaded versus GPU-enhanced
performance would be the fair comparison.
