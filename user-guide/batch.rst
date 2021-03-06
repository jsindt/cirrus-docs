Running Jobs on Cirrus
======================

As with most HPC services, Cirrus uses a scheduler to manage access to
resources and ensure that the thousands of different users of system
are able to share the system and all get access to the resources they
require. Cirrus uses the Slurm software to schedule jobs.

Writing a submission script is typically the most convenient way to
submit your job to the scheduler. Example submission scripts
(with explanations) for the most common job types are provided below.

Interactive jobs are also available and can be particularly useful for
developing and debugging applications. More details are available below.

.. hint::

  If you have any questions on how to run jobs on Cirrus do not hesitate
  to contact the `Cirrus Service Desk <mailto:support@cirrus.ac.uk>`_.

You typically interact with Slurm by issuing Slurm commands
from the login nodes (to submit, check and cancel jobs), and by
specifying Slurm directives that describe the resources required for your
jobs in job submission scripts.


Basic Slurm commands
--------------------

There are three key commands used to interact with the Slurm on the
command line:

-  ``sinfo`` - Get information on the partitions and resources available
-  ``sbatch jobscript.slurm`` - Submit a job submission script (in this case called: ``jobscript.slurm``) to the scheduler
-  ``squeue`` - Get the current status of jobs submitted to the scheduler
-  ``scancel 12345`` - Cancel a job (in this case with the job ID ``12345``)

We cover each of these commands in more detail below.

``sinfo``: information on resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``sinfo`` is used to query information about available resources and partitions.
Without any options, ``sinfo`` lists the status of all resources and partitions,
e.g.

::

   [auser@cirrus-login3 ~]$ sinfo 

   PARTITION   AVAIL  TIMELIMIT  NODES  STATE NODELIST 
   standard       up   infinite     36   idle r1i1n[0-35] 
   standard       up   infinite    244   down r1i0n[0-35],r1i2n[0-35],r1i3n[0-35],r1i4n[0-35],r1i5n[0-35],r1i6n[0-35],r1i7n[0-6,9-15,18-24,27-33] 
   gpu-skylake    up   infinite      2   down r2i3n[0-1] 

``sbatch``: submitting jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``sbatch`` is used to submit a job script to the job submission system. The script
will typically contain one or more ``srun`` commands to launch parallel tasks.

When you submit the job, the scheduler provides the job ID, which is used to identify
this job in other Slurm commands and when looking at resource usage in SAFE.

::

  [auser@cirrus-login3 ~]$ sbatch test-job.slurm
  Submitted batch job 12345

``squeue``: monitoring jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``squeue`` without any options or arguments shows the current status of all jobs
known to the scheduler. For example:

::

  [auser@cirrus-login3 ~]$ squeue
            JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
            1554  comp-cse CASTEP_a  auser  R       0:03      2 r2i0n[18-19] 

will list all jobs on Cirrus.

The output of this is often overwhelmingly large. You can restrict the output
to just your jobs by adding the ``-u $USER`` option:

::

  [auser@cirrus-login3 ~]$ squeue -u $USER

``scancel``: deleting jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~

``scancel`` is used to delete a jobs from the scheduler. If the job is waiting 
to run it is simply cancelled, if it is a running job then it is stopped 
immediately. You need to provide the job ID of the job you wish to cancel/stop.
For example:

::

  [auser@cirrus-login3 ~]$ scancel 12345

will cancel (if waiting) or stop (if running) the job with ID ``12345``.

Resource Limits
---------------

There are different resource limits on Cirrus for different purposes.

.. note::

   Details on the resource limits will be added shortly.

.. TODO: Add in partition and QOS limits once they are known

Troubleshooting
---------------

Slurm error messages
~~~~~~~~~~~~~~~~~~~~

Sometimes Slurm will return an error when a job is submitted. The following is a list of common
errors and how to fix them.

* error: Unable to allocate resources: Invalid account or account/partition combination specified
* error: Unable to allocate resources: User's group not permitted to use this partition

  * You must use a valid account, partition and QoS combination.

* error: Unable to allocate resources: No partition specified or system default partition
* error: invalid partition specified: <partition_name>
* error: Unable to allocate resources: Invalid partition name specified

  * You must use a valid partition. Add "--partition=PARTITION_NAME" to your submission script.

* error: Unable to allocate resources: Invalid qos specification

  * You must use a valid QoS. Add "--qos=QOS_NAME" to your submission script.

* Requested partition configuration not available now

  * The number of nodes/cores requested is not available.

* error: unrecognized option <option>

  * One of your options is invalid or has a typo.

* error: Unable to allocate resources: Requested time limit is invalid (missing or exceeds some limit)
* error: --time limit option required

  * The time limit of your script is either missing or is too long. Add "--time=minutes" to your submission script.



Slurm queued reasons
~~~~~~~~~~~~~~~~~~~~

The ``squeue`` command allows users to view information for jobs managed by Slurm. Jobs
typically go through the following states: PENDING, RUNNING, COMPLETING, and COMPLETED.
The first table provides a description of some job state codes. The second table provides a description
of the reasons that cause a job to be in a state.

.. list-table:: Slurm Job State codes
   :widths: 20 10 70
   :header-rows: 1

   * - Status
     - Code
     - Description
   * - PENDING
     - PD
     - Job is awaiting resource allocation.
   * - RUNNING
     - R
     - Job currently has an allocation.
   * - SUSPENDED
     - S
     - Job currently has an allocation.
   * - COMPLETING
     - CG
     - Job is in the process of completing. Some processes on some nodes may still be active.
   * - COMPLETED
     - CD
     - Job has terminated all processes on all nodes with an exit code of zero.
   * - TIMEOUT
     - TO
     - Job terminated upon reaching its time limit.
   * - STOPPED
     - ST
     - Job has an allocation, but execution has been stopped with SIGSTOP signal. CPUS have been retained by this job.
   * - OUT_OF_MEMORY
     - OOM
     - Job experienced out of memory error.
   * - FAILED
     - F
     - Job terminated with non-zero exit code or other failure condition.
   * - NODE_FAIL
     - NF
     - Job terminated due to failure of one or more allocated nodes.
   * - CANCELLED
     - CA
     - Job was explicitly cancelled by the user or system administrator. The job may or may not have been initiated.

For a full list of see `Job State Codes <https://slurm.schedmd.com/squeue.html#lbAG>`__

.. list-table:: Slurm Job Reasons
   :widths: 30 70
   :header-rows: 1

   * - Reason
     - Description
   * - Priority
     - One or more higher priority jobs exist for this partition or advanced reservation. 
   * - Resources
     - The job is waiting for resources to become available. 
   * - BadConstraints
     - The job's constraints can not be satisfied. 
   * - BeginTime
     - The job's earliest start time has not yet been reached. 
   * - Dependency
     - This job is waiting for a dependent job to complete. 
   * - Licenses
     - The job is waiting for a license. 
   * - WaitingForScheduling
     - No reason has been set for this job yet. Waiting for the scheduler to determine the appropriate reason. 
   * - Prolog
     - Its PrologSlurmctld program is still running. 
   * - JobHeldAdmin
     - The job is held by a system administrator. 
   * - JobHeldUser
     - The job is held by the user. 
   * - JobLaunchFailure
     - The job could not be launched. This may be due to a file system problem, invalid program name, etc. 
   * - NonZeroExitCode
     - The job terminated with a non-zero exit code. 
   * - InvalidAccount
     - The job's account is invalid.
   * - InvalidQOS
     - The job's QOS is invalid. 
   * - QOSUsageThreshold
     - Required QOS threshold has been breached. 
   * - QOSJobLimit
     - The job's QOS has reached its maximum job count. 
   * - QOSResourceLimit
     - The job's QOS has reached some resource limit. 
   * - QOSTimeLimit
     - The job's QOS has reached its time limit. 
   * - NodeDown
     - A node required by the job is down. 
   * - TimeLimit
     - The job exhausted its time limit. 
   * - ReqNodeNotAvail
     - Some node specifically required by the job is not currently available. The node may currently be in use, reserved for another job, in an advanced reservation, DOWN, DRAINED, or not responding. Nodes which are DOWN, DRAINED, or not responding will be identified as part of the job's "reason" field as "UnavailableNodes". Such nodes will typically require the intervention of a system administrator to make available. 

For a full list of see `Job Reasons <https://slurm.schedmd.com/squeue.html#lbAF>`__

Output from Slurm jobs
----------------------

Slurm places standard output (STDOUT) and standard error (STDERR) for each
job in the file ``slurm_<JobID>.out``. This file appears in the
job's working directory once your job starts running.

.. note::

  This file is plain text and can contain useful information to help debugging
  if a job is not working as expected. The Cirrus Service Desk team will often
  ask you to provide the contents of this file if oyu contact them for help 
  with issues.

Specifying resources in job scripts
-----------------------------------

You specify the resources you require for your job using directives at the
top of your job submission script using lines that start with the directive
``#SBATCH``. 

.. note::

  Options provided using ``#SBATCH`` directives can also be specified as 
  command line options to ``srun``.

If you do not specify any options, then the default for each option will
be applied. As a minimum, all job submissions must specify the budget that
they wish to charge the job too with the option:

  - ``--account=<budgetID>`` your budget ID is usually something like
    ``t01`` or ``t01-test``. You can see which budget codes you can 
    charge to in SAFE.

Other common options that are used are:

  - ``--time=<hh:mm:ss>`` the maximum walltime for your job. *e.g.* For a 6.5 hour
    walltime, you would use ``--time=6:30:0``.
  - ``--job-name=<jobname>`` set a name for the job to help identify it in 
    Slurm command output.

In addition, parallel jobs will also need to specify how many nodes,
parallel processes and threads they require.

  - ``--exclusive`` to ensure that you have exclusive access to a compute node
  - ``--nodes=<nodes>`` the number of nodes to use for the job.
  - ``--tasks-per-node=<processes per node>`` the number of parallel processes
    (e.g. MPI ranks) per node.
  - ``--cpus-per-task=<threads per task>`` the number of threads per
    parallel process (e.g. number of OpenMP threads per MPI task for
    hybrid MPI/OpenMP jobs). **Note:** you must also set the ``OMP_NUM_THREADS``
    environment variable if using OpenMP in your job.

.. note::

  For parallel jobs, you should request exclusive node access with the
  ``--exclusive`` option to ensure you get the expected resources and
  performance.

``srun``: Launching parallel jobs
---------------------------------

If you are running parallel jobs, your job submission script should contain
one or more ``srun`` commands to launch the parallel executable across the
compute nodes. As well as launching the executable, ``srun`` also allows you
to specify the distribution and placement (or *pinning*) of the parallel
processes and threads.

.. note::

  Usually, you will want to use ``srun`` without any additional options. Used
  in this way, ``srun`` will use the options you provided to ``sbatch`` (or ``salloc``, see below)
  to distribute the parallel tasks.

Example job submission scripts
-------------------------------

A subset of example job submission scripts are included in full below.

Example: job submission script for OpenMP parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A simple OpenMP job submission script to submit a job using 1 compute
nodes and 36 threads for 20 minutes would look like:

::

    #!/bin/bash

    # Slurm job options (name, compute nodes, job time)
    #SBATCH --job-name=Example_OpenMP_Job
    #SBATCH --time=0:20:0
    #SBATCH --exclusive
    #SBATCH --nodes=1
    #SBATCH --tasks-per-node=1
    #SBATCH --cpus-per-task=36

    # Replace [budget code] below with your budget code (e.g. t01)
    #SBATCH --account=[budget code]             

    # Set the number of threads to the CPUs per task
    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

    # Launch the parallel job
    #   Using 36 threads per node
    #   srun picks up the distribution from the sbatch options
    srun --cpu-bind=cores ./my_openmp_executable.x

This will run your executable "my\_openmp\_executable.x" in parallel on 36 threads. Slurm will
allocate 1 node to your job and srun will place 36 threads (one per physical core).

See above for a more detailed discussion of the different ``sbatch`` options


Example: job submission script for MPI parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A simple MPI job submission script to submit a job using 4 compute
nodes and 128 MPI ranks per node for 20 minutes would look like:

::

    #!/bin/bash

    # Slurm job options (name, compute nodes, job time)
    #SBATCH --job-name=Example_MPI_Job
    #SBATCH --time=0:20:0
    #SBATCH --exclusive
    #SBATCH --nodes=4
    #SBATCH --tasks-per-node=36
    #SBATCH --cpus-per-task=1

    # Replace [budget code] below with your budget code (e.g. t01)
    #SBATCH --account=[budget code]             

    # Set the number of threads to 1
    #   This prevents any threaded system libraries from automatically 
    #   using threading.
    export OMP_NUM_THREADS=1

    # Launch the parallel job
    #   Using 144 MPI processes and 128 MPI processes per node
    #   srun picks up the distribution from the sbatch options
    srun ./my_mpi_executable.x

This will run your executable "my\_mpi\_executable.x" in parallel on 144
MPI processes using 4 nodes (36 cores per node, i.e. not using hyper-threading). Slurm will
allocate 4 nodes to your job and srun will place 36 MPI processes on each node
(one per physical core).

See above for a more detailed discussion of the different ``sbatch`` options

Example: job submission script for MPI+OpenMP (mixed mode) parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mixed mode codes that use both MPI (or another distributed memory
parallel model) and OpenMP should take care to ensure that the shared
memory portion of the process/thread placement does not span more than
one node. This means that the number of shared memory threads should be
a factor of 36.

In the example below, we are using 4 nodes for 6 hours. There are 8 MPI
processes in total (2 MPI processes per node) and 18 OpenMP threads per MPI
process. This results in all 36 physical cores per node being used.

.. note:: 

   the use of the ``--cpu-bind=cores`` option to generate the correct 
   affinity settings.

::

    #!/bin/bash

    # Slurm job options (name, compute nodes, job time)
    #SBATCH --name=Example_MPI_Job
    #SBATCH --time=0:20:0
    #SBATCH --exclusive
    #SBATCH --nodes=4
    #SBATCH --ntasks=8
    #SBATCH --tasks-per-node=2
    #SBATCH --cpus-per-task=18

    # Replace [budget code] below with your project code (e.g. t01)
    #SBATCH --account=[budget code] 

    # Set the number of threads to 18
    #   There are 18 OpenMP threads per MPI process
    export OMP_NUM_THREADS=18

    # Launch the parallel job
    #   Using 8 MPI processes
    #   2 MPI processes per node
    #   18 OpenMP threads per MPI process
 
   srun --cpu-bind=cores ./my_mixed_executable.x arg1 arg2

Job arrays
----------

The Slurm job scheduling system offers the *job array* concept,
for running collections of almost-identical jobs. For example,
running the same program several times with different arguments
or input data.

Each job in a job array is called a *subjob*.  The subjobs of a job
array can be submitted and queried as a unit, making it easier and
cleaner to handle the full set, compared to individual jobs.

All subjobs in a job array are started by running the same job script.
The job script also contains information on the number of jobs to be
started, and Slurm provides a subjob index which can be passed to
the individual subjobs or used to select the input data per subjob.

Job script for a job array
~~~~~~~~~~~~~~~~~~~~~~~~~~

As an example, the following script runs 56 subjobs, with the subjob
index as the only argument to the executable. Each subjob requests a
single node and uses all 36 cores on the node by placing 1 MPI 
process per core and specifies 4 hours maximum runtime per subjob:

::

    #!/bin/bash
    # Slurm job options (name, compute nodes, job time)
    #SBATCH --name=Example_Array_Job
    #SBATCH --time=0:20:0
    #SBATCH --exclusive
    #SBATCH --nodes=4
    #SBATCH --tasks-per-node=36
    #SBATCH --cpus-per-task=1
    #SBATCH --array=0-55

    # Replace [budget code] below with your budget code (e.g. t01)
    #SBATCH --account=[budget code]  

    # Set the number of threads to 1
    #   This prevents any threaded system libraries from automatically 
    #   using threading.
    export OMP_NUM_THREADS=1

    srun /path/to/exe $SLURM_ARRAY_TASK_ID


Submitting a job array
~~~~~~~~~~~~~~~~~~~~~~

Job arrays are submitted using ``sbatch`` in the same way as for standard
jobs:

::

    sbatch job_script.pbs

Job chaining
------------

Job dependencies can be used to construct complex pipelines or chain together long
simulations requiring multiple steps.

.. note::

   The ``--parsable`` option to ``sbatch`` can simplify working with job dependencies.
   It returns the job ID in a format that can be used as the input to other 
   commands.

For example:

::

   jobid=$(sbatch --parsable first_job.sh)
   sbatch --dependency=afterok:$jobid second_job.sh

or for a longer chain:

::

   jobid1=$(sbatch --parsable first_job.sh)
   jobid2=$(sbatch --parsable --dependency=afterok:$jobid1 second_job.sh)
   jobid3=$(sbatch --parsable --dependency=afterok:$jobid1 third_job.sh)
   sbatch --dependency=afterok:$jobid2,afterok:$jobid3 last_job.sh

Interactive Jobs: ``salloc``
----------------------------

When you are developing or debugging code you often want to run many
short jobs with a small amount of editing the code between runs. This
can be achieved by using the login nodes to run MPI but you may want
to test on the compute nodes (e.g. you may want to test running on 
multiple nodes across the high performance interconnect). One of the
best ways to achieve this on Cirrus is to use interactive jobs.

An interactive job allows you to issue ``srun`` commands directly
from the command line without using a job submission script, and to
see the output from your program directly in the terminal.

You use the ``salloc`` command to reserve compute nodes for interactive
jobs.

To submit a request for an interactive job reserving 8 nodes
(1024 physical cores) for 1 hour you would
issue the following qsub command from the command line:

::

    salloc --exclusive --nodes=2 --tasks-per-node=36 --cpus-per-task=1 --time=1:0:0 --account=t01
    

When you submit this job your terminal will display something like:

::

    salloc: Granted job allocation 24236
    salloc: Waiting for resource configuration
    salloc: Nodes cn13 are ready for job

It may take some time for your interactive job to start. Once it
runs you will enter a standard interactive terminal session.
Whilst the interactive session lasts you will be able to run parallel
jobs on the compute nodes by issuing the ``srun``  command
directly at your command prompt using the same syntax as you would inside
a job script. The maximum number of nodes you can use is limited by resources
requested in the ``salloc`` command.

If you know you will be doing a lot of intensive debugging you may
find it useful to request an interactive session lasting the expected
length of your working session, say a full day.

Your session will end when you hit the requested walltime. If you
wish to finish before this you should use the ``exit`` command - this will
return you to your prompt before you issued the ``salloc`` command.

Reservations
------------

The mechanism for submitting reservations on Cirrus has yet to be specified.

.. TODO: Add information on how to submit reservations
