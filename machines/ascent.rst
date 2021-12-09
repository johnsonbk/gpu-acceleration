######
Ascent
######

MPI issue
=========

``test_get_close_obs`` fails to compile on Ascent because the
``test_get_close_obs_nml`` namelist input statement doesn't exactly match the
namelist in ``DART-hackathon21/hackathon/get_close_obs/work/input.nml``.

Since ``input.nml`` was altered to include a tolerance:

.. code-block:: fortran

   &test_get_close_obs_nml
      ...
      tolerance = 0.00000001
      ...
   /

Lines 95-96 in ``test_get_close_obs.f90`` bust be altered to:

.. code-block::

   namelist /test_get_close_obs_nml/ my_num_obs, obs_to_assimilate, num_repeats, lon_start, lon_end, &
                                   lat_start, lat_end,cutoff,compare_to_correct,tolerance

