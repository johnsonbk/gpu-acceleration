######
Nsight
######

Nvdia's general-purpose profiling tool is called
`Nsight Systems <https://docs.nvidia.com/nsight-systems/UserGuide/index.html>`_.
Bob Knight, John Stone and Daniel Horowitz deliver a lecture on `Nvidia Nsight
Systems <https://www.nvidia.com/en-us/on-demand/session/gtcsiliconvalley2018-s8718/>`_.

The tutorial posted in the hackathon's #announcement Slack channel is
delivered by `Max Katz of Nvidia <https://drive.google.com/file/d/1TEPiRpxqZXK2iqzy1uAQoAlrH3u7z-iX/view?usp=sharing>`_.

There are two additional tools within the Nsight product family:

- *Nsight Compute*, which is used for CUDA
- *Nsight Graphics*, which is used for graphics shading tools

The latter are multipass tuners that are great for specific applications.

Two ways to use Nsight systems
==============================

Graphical user interface
------------------------

There is a GUI that can be used via a host-target set up.

Command line interface
----------------------

When the ``nvhpc`` compiler is loaded, nsight systems can be called from the
command line using:

.. code-block::

   nsys [command_switch][optional command_switch_options][application] [optional application_options]


