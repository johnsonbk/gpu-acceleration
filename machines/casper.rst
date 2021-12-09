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
   module load nvhpc/20.11
   module load ncarcompilers/0.5.0
   module load openmpi/4.1.1
   module load netcdf
   module list

.. note::

   There is some murmuring among the mentors that some of the difficulty we're
   experiencing on Casper is merely because the 20.11 release of ``nvhpc`` is
   old.

Job script
==========

The working job script is here:

.. code-block::

   /glade/scratch/criedel/HACKATHON/DART-hackathon21/hackathon/get_close_obs/work/casper_submit.sh

