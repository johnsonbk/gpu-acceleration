####
NVTX
####

The Nvidia tools extension SDK (NVTX) is an API that allows for annotating code
for performance evaluation with Nsight. `NVTX has a Fortran interface <https://docs.nvidia.com/hpc-sdk/compilers/fortran-cuda-interfaces/index.html#cfnvtx-runtime>`_. 

The ``-lnvhpcwrapnvtx`` flag must be added to ``mkmf.template`` in order for it
to work properly:

.. code-block::

   ACCFLAGS = -acc -ta=tesla:cc70,deepcopy -Minfo=accel -I${NVHPC_ROOT_PATH}/include -lnvhpcwrapnvtx -Minstrument

The ``-Minstrument`` flag is optional, however it supposedly inserts NVTX
ranges at subprogram entries and exits.

Within Fortran source code, the ``nvtx`` module must be used:

.. code-block:: fortran

   use nvtx

and ranges can be started and ended as demonstrated below.

.. code-block:: fortran

   call nvtxStartRange("First label")
 
   !$acc parallel loop reduction(+:tmp1,tmp2,tmp3)

   GLOBAL_OBS: do obs = 1, num_obs_to_assimilate

   ...

   enddo GLOBAL_OBS     
    
   !$acc end parallel   
                      
   call nvtxEndRange

