Running Jobs on Cirrus
======================

The Cirrus facility uses PBSPro to schedule jobs.

Writing a submission script is typically the most convenient way to
submit your job to the job submission system. Example submission scripts
(with explanations) for the most common job types are provided below.

Interactive jobs are also available and can be particularly useful for
developing and debugging applications. More details are available below.

If you have any questions on how to run jobs on Cirrus do not hesitate
to contact the EPCC Helpdesk.

Using PBS Pro
-------------

You typically interact with PBS by (1) specifying PBS directives in job
submission scripts (see examples below) and (2) issuing PBS commands
from the login nodes.

There are three key commands used to interact with the PBS on the
command line:

-  ``qsub``
-  ``qstat``
-  ``qdel``

Check the PBS ``man`` page for more advanced commands:

::

    man pbs

The qsub command
~~~~~~~~~~~~~~~~

The qsub command submits a job to PBS:

::

    qsub job_script.pbs

This will submit your job script "job\_script.pbs" to the job-queues.
See the sections below for details on how to write job scripts.


The qstat command
~~~~~~~~~~~~~~~~~

Use the command qstat to view the job queue. For example:

::

    qstat

will list all jobs on Cirrus.

You can view just your jobs by using:

::

    qstat -u $USER

The ``-a`` option to qstat provides the output in a more useful
format.

To see more information about a queued job, use:

::

    qstat -f $JOBID

This option may be useful when your job fails to enter a running state.
The output contains a PBS ``comment`` field which may explain why the job
failed to run.


The qdel command
~~~~~~~~~~~~~~~~

Use this command to delete a job from Cirrus's job queue. For example:

::

    qdel $JOBID

will remove the job with ID ``$JOBID`` from the queue.

Output from PBS jobs
--------------------

PBS produces standard output and standard error for each batch job can
be found in files ``<jobname>.o<Job ID>`` and ``<jobname>.e<Job ID>``
respectively. These files appear in the job's working directory once
your job has completed or its maximum allocated time to run (i.e. wall
time, see later sections) has ran out.

Running Parallel Jobs
---------------------

This section describes how to write job submission scripts specifically
for different kinds of parallel jobs on Cirrus.

All parallel job submission scripts require (as a minimum) you to
specify three things:

-  The number of virtual cores you require via the
   ``-l select=[cores]`` option. Each node has 36 physical
   cores (2x 18-core sockets) but hyper-threads are enabled (2 per core).
   Thus, to request 2 nodes, for instance, you must select 144 "cores"
   (36 cores \* 2 hyper-threads \* 2 nodes).
-  The maximum length of time (i.e. walltime) you want the job to run
   for via the ``-l walltime=[hh:mm:ss]`` option. To ensure the
   minimum wait time for your job, you should specify a walltime as
   short as possible for your job (i.e. if your job is going to run for
   3 hours, do not specify 12 hours). On average, the longer the
   walltime you specify, the longer you will queue for.
-  The project code that you want to charge the job to via the
   ``-A [project code]`` option

In addition to these mandatory specifications, there are many other
options you can provide to PBS. The following options may be useful:

- By default compute nodes are shared, meaning other jobs may be placed
  alongside yours if your resource request (with -l select) leaves some
  cores free. To guarantee exclusive node usage, use the option ``-l place=excl``.
- The name for your job is set using ``-N My_job``. In the examples below
  the name will be "My\_job", but you can replace "My\_job" with any
  name you want. The name will be used in various places. In particular
  it will be used in the queue listing and to generate the name of your
  output and/or error file(s). Note there is a limit on the size of the
  name.


Parallel job launcher ``mpiexec_mpt``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The job launcher for parallel jobs on Cirrus is ``mpiexec_mpt``.

**Note:** the parallel job launcher is only available once you have
loaded the ``mpt`` module.

A sample MPI job launch line using ``mpiexec_mpt`` looks like:

::

    mpiexec_mpt -n 72 -ppn 36 ./my_mpi_executable.x arg1 arg2

This will start the parallel executable "my\_mpi\_executable.x" with
arguments "arg1" and "arg2". The job will be started using 72 MPI
processes, with 36 MPI processes are placed on each compute node 
(this would use all the physical cores on each node).

The most important ``mpiexec_mpt`` flags are:

 ``-n [total number of MPI processes]``
    Specifies the total number of distributed memory parallel processes
    (not including shared-memory threads). For jobs that use all
    physical cores this will usually be a multiple of 36. The default on
    Cirrus is 1.
 ``-ppn [parallel processes per node]``
    Specifies the number of distributed memory parallel processes per
    node. There is a choice of 1-36 for physical cores on Cirrus compute
    nodes (1-72 if you are using Hyper-Threading) If you are running with
    exclusive node usage, the most economic choice is always to run with
    "fully-packed" nodes on all physical cores if possible, i.e.
    ``-N 36`` . Running "unpacked" or "underpopulated" (i.e. not using
    all the physical cores on a node) is useful if you need large
    amounts of memory per parallel process or you are using more than
    one shared-memory thread per parallel process.

If you are running hybrid MPI/OpenMP code you will also often make
use of the ``omplace`` tool in your job launcher line. This tool 
takes the number of threads as the option ``-nt``:

 ``-nt [threads per parallel process]``
    Specifies the number of cores for each parallel process to use for
    shared-memory threading. (This is in addition to the
    ``OMP_NUM_THREADS`` environment variable if you are using OpenMP for
    your shared memory programming.) The default on Cirrus is 1.


Please use ``man mpiexec_mpt`` and ``man omplace`` to query further options.
(Again, these are only available once you have loaded the ``mpt`` module.)

Example: job submission script for MPI parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A simple MPI job submission script to submit a job using 2 compute
nodes (maximum of 72 physical cores) for 20 minutes would look like:

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_MPI_Job
    # To get two nodes we need 72*2 = 144 cores (hyper-threading included)
    #PBS -l select=144
    #PBS -l walltime=00:20:00

    # To get exclusive node usage
    #PBS -l place=excl

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]             

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR
  
    # Load any required modules
    module load mpt
    module load intel-compilers-16

    # Set the number of threads to 1
    #   This prevents any threaded system libraries from automatically 
    #   using threading.
    export OMP_NUM_THREADS=1

    # Launch the parallel job
    #   Using 72 MPI processes and 24 MPI processes per node
    mpiexec_mpt -n 72 -ppn 36 ./my_mpi_executable.x arg1 arg2 > my_stdout.txt 2> my_stderr.txt

This will run your executable "my\_mpi\_executable.x" in parallel on 72
MPI processes using 2 nodes (36 cores per node, i.e. not using hyper-threading). PBS will
allocate 2 nodes to your job and mpirun_mpt will place 36 MPI processes on each node
(one per physical core).

See above for a detailed discussion of the different PBS options

Example: job submission script for MPI+OpenMP (mixed mode) parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mixed mode codes that use both MPI (or another distributed memory
parallel model) and OpenMP should take care to ensure that the shared
memory portion of the process/thread placement does not span more than
one node. This means that the number of shared memory threads should be
a factor of 18.

In the example below, we are using 2 nodes for 6 hours. There are 4 MPI
processes in total and 18 OpenMP threads per MPI process. Note the use
of the ``omplace`` command to specify the number of threads.

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_MixedMode_Job
    #PBS -l select=144
    #PBS -l walltime=6:0:0

    # To get exclusive node usage
    #PBS -l place=excl

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR

    # Load any required modules
    module load mpt
    module load intel-compilers-16

    # Set the number of threads to 18
    #   There are 18 OpenMP threads per MPI process
    export OMP_NUM_THREADS=18

    # Launch the parallel job
    #   Using 4 MPI processes
    #   2 MPI processes per node
    #   18 OpenMP threads per MPI process
    mpiexec_mpt -n 4 -ppn 2 omplace -nt 18 ./my_mixed_executable.x arg1 arg2 > my_stdout.txt 2> my_stderr.txt

Example: job submission script for parallel non-MPI based jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to run on multiple nodes, where each node is running a self-contained job, not using MPI
(e.g.) for processing data or a parameter sweep, you can use the mpiexec_mpt launcher to control job placement.

In the example script below, work.bash is a bash script which runs a threaded executable with a command-line input and
perf.bash is a bash script which copies data from the CPU performance counters to an output file. As both handle the
threading themselves, it is sufficient to allocate 1 MPI rank. Using the ampersand "&" allows both to execute simultaneously.
Both work.bash and perf.bash run on 2 nodes.

::

   #!/bin/bash --login
   # PBS job options (name, compute nodes, job time)
   #PBS -N Example_MixedMode_Job
   #PBS -l select=144
   #PBS -l walltime=6:0:0
   
   # To get exclusive node usage
   #PBS -l place=excl
   
   # Replace [budget code] below with your project code (e.g. t01)
   #PBS -A [budget code]
   
   # Change to the directory that the job was submitted from
   cd $PBS_O_WORKDIR
   
   # Load any required modules
   module load mpt

   # Set this variable to inform mpiexec_mpt these are not MPI jobs
   export MPI_SHEPHERD=true

   # Execute work and perf scripts on nodes simultaneously.
   mpiexec_mpt -n 2 -ppn 1 work.bash &
   mpiexec_mpt -n 2 -ppn 1 perf.bash &
   wait

**Note:** the wait command is required to stop the PBS job finishing before the scripts finish.
If you find odd behaviour, especially with respect to the values of bash variables, double check you
have set MPI_SHEPHERD=true

MPI on the login nodes
~~~~~~~~~~~~~~~~~~~~~~

If you want to run a short interactive parallel applications (e.g. for 
debugging) then you can run compiled MPI applications on the login nodes.

For instance, to run a simple, short 4-way MPI job on the login node, issue the
following command (once you have loaded the appropriate modules):

:: 

    mpirun -n 4 ./hello_mpi.x

**Note:** you should not run long, compute- or memory-intensive jobs on the 
login nodes. Any such processes a liable to termination by the system
with no warning.

Serial Jobs
-----------

Serial jobs are setup in a similar way to parallel jobs on Cirrus. The
only changes are:

1. You should request a single core with ``select=1``
2. You will not need to use a parallel job launcher to run your executable

A simple serial script to compress a file would be:

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_Serial_Job
    #PBS -l select=1
    #PBS -l walltime=0:20:0

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR

    # Load any required modules
    module load intel-compilers-16

    # Set the number of threads to 1 to ensure serial
    export OMP_NUM_THREADS=1

    # Run the serial executable
    gzip my_big_file.dat

Interactive Jobs
----------------

When you are developing or debugging code you often want to run many
short jobs with a small amount of editing the code between runs. This
can be achieved by using the login nodes to run MPI but you may want
to test on the compute nodes (e.g. you may want to test running on 
multiple nodes across the high performance interconnect). One of the
best ways to achieve this on Cirrus is to use interactive jobs.

An interactive job allows you to issue ``mpirun_mpt`` commands directly
from the command line without using a job submission script, and to
see the output from your program directly in the terminal.

To submit a request for an interactive job reserving 8 nodes
(288 physical cores, 576 hyper-threaded cores)) for 1 hour you would
issue the following qsub command from the command line:

::

    qsub -IVl select=576,walltime=1:0:0 -A [project code]

When you submit this job your terminal will display something like:

::

    qsub: waiting for job 19366.indy2-login0 to start

It may take some time for your interactive job to start. Once it
runs you will enter a standard interactive terminal session.
Whilst the interactive session lasts you will be able to run parallel
jobs on the compute nodes by issuing the ``mpirun_mpt``  command
directly at your command prompt (remember you will need to load the
``mpt`` module and any compiler modules before running)  using the
same syntax as you would inside a job script. The maximum number
of cores you can use is limited by the value of select you specify
when you submit a request for the interactive job.

If you know you will be doing a lot of intensive debugging you may
find it useful to request an interactive session lasting the expected
length of your working session, say a full day.

Your session will end when you hit the requested walltime. If you
wish to finish before this you should use the ``exit`` command.

Debugging Jobs
-------------

Allinea's Forge tool suite is installed on Cirrus and DDT,  which is a debugging tool for scalar, multi-threaded and large-scale parallel applications, is available for use in debugging your codes. To compile your code for debugging you will usually want to specify the ``-O0`` option to turn off all code optimisation (as this can produce a mismatch between source code line numbers and debugging information) and ``-g`` to include debugging information in the compiled executable. To use this package you will need to load the Allinea Forge module and execute ``forge``:

::

    module load allinea
    forge

Debugging runs on the login nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can execute and debug your MPI code on the login node which is useful for immediate development work with short, simple runs to avoid having to wait in the queue. Firstly ensure you have loaded the ``mpt`` library, then start forge and click *Run*. Fill in the nescesary details of your code under the *application* pane, then check the MPI tick box, specify the number of MPI processes you wish to run and ensure implementation is set to *SGI MPT (2.10+)*. If this is not set correctly then you can update the configuration via clicking the *Change* button and selecting this option on the *MPI/UPC Implementation* field of the system pane. When you are happy with this hit *Run* to start.

Debugging runs on the compute nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This involves DDT submitting your job to the queue, this then running and as soon as the compute nodes start executing you will drop into the debug session and be able to interact with your code. Similarly to running on the login node, fill in details of your application and ensure that MPI is ticked. But now change the implementation from *SGI MPT (2.10+)* to *SGI MPT (2.10+, batch)* as we are running via the batch system. Then check the *Submit to Queue* tick box and click the *Configure* button. In the settings window that pops up you can specify the submission template, one has been prepared one for Cirrus at ``/lustre/sw/allinea/forge-7.0.0/templates/cirrus.qtf`` which we suggest you use - although you are very free to chose another one and/or specialise this as you require. Back on the run page, click the *Parameters* button and fill in the maximum wallclock time, the budget to charge to and the total number of virtual cores required which determine the number of nodes and are provided as an argument to the *-l select=* PBS option. Back on the *run* dialog ensure look at the *MPI* pane, ensure the *number of processes* and *processes per node* settings are correct and then hit *Submit*.

Memory debugging with DDT
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are dynamically linking your code and debugging it on the login node then this is fine (just ensure that the *Preload the memory debugging library* option is ticked in the *Details* pane.) If you are dynamically linking but intending to debug running on the compute nodes, or statically linking then you need to include the compile option ``-Wl,--allow-multiple-definition`` and explicitly link your executable with Allinea's memory debugging library. The exactly library to link against depends on your code; ``-ldmalloc`` (for no threading with C), ``-ldmallocth`` (for threading with C), ``-ldmallocxx`` (for no threading with C++) or ``-ldmallocthcxx`` (for threading with C++). The library locations are all set up when the *allinea* module is loaded so these libraries should be found without further arguments.

Getting further help on DDT
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `DDT website <http://www.allinea.com/products/ddt/>`__
-  `DDT support page <https://www.allinea.com/get-support>`__
-  `DDT user guide <https://www.allinea.com/user-guide/forge/userguide.html>`__
