######
Casper
######


Working set of modules
======================

Chris and the team were able to configure the modules correctly to build the
``get_close`` kernel on Casper. The working build script is here:

.. code-block::

   /glade/scratch/criedel/HACKATHON/DART-hackathon21/hackathon/get_close_obs/work/build_testcode.sh 

The working modules commands are:

.. code-block::

   module purge
   module load ncarenv/1.3
   module load nvhpc/21.9
   module load ncarcompilers/0.5.0
   module load openmpi/4.1.1
   module load netcdf
   module list

.. note::

   The default ``nvhpc`` module on Casper is ``20.11``. It doesn't support 
   NVTX, so use ``21.9`` instead.

Interactive job
===============

This command works for requesting an interactive job on Casper:

.. code-block::

   execcasper -A P86850054 -q gpudev -l select=1:ncpus=8:ngpus=4:mpiprocs=8:mem=200GB -l walltime=00:30:00


Interactive build
=================

.. code-block::

   execcasper -A P86850054 -q gpudev -l select=1:ncpus=8:ngpus=4:mpiprocs=8:mem=200GB -l walltime=00:30:00
   cd /glade/work/johnsonb/git/DART-hackathon21/hackathon/get_close_obs/work/
   ./build_testcode.sh 
   ./test_get_close_obs

Job script
==========

The working job script is here:

.. code-block::

   /glade/scratch/criedel/HACKATHON/DART-hackathon21/hackathon/get_close_obs/work/casper_submit.sh

