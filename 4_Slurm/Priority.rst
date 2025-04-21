Priority and Usage Calculation
##############################

Rockfish uses a multi-factor job priority system to determine which jobs run first. These factors work together to ensure equitable access while keeping the cluster as busy and efficient as possible. Understanding these can help users predict queue behavior and optimize job placement.

How Job Priority is Calculated
==============================

Slurm evaluates each pending job with a weighted formula combining the following components:

- **Fairshare**: Each PI or group is assigned a share based on their quarterly allocation (in core hours). Slurm tracks usage over time and adjusts each group's priority relative to their share. Groups that have recently used less of their allocation get higher priority, and those that have consumed more see lower scores. This promotes equitable access.
- **Job Age**: Jobs gain priority the longer they wait in the queue.
- **Job Size**: Smaller jobs receive a small boost, particularly to improve backfilling opportunities — the ability for the scheduler to fit small jobs into idle gaps in hardware.
- **Partition and QOS Priority**: Some partitions or QOS configurations may be assigned static priority tiers or usage caps.

To inspect job priority components, use the `sprio` command:

.. code-block:: bash

   sprio -j <jobid>
   sprio -u <user>

   [root@login01 ~]# sprio -u user123
             JOBID PARTITION     USER   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE  PARTITION        QOS
            111111 parallel   user123      21904          0       1423        489      19990          1          0
            111112 parallel   user123      21904          0       1423        489      19990          1          0
            111113 parallel   user123      21904          0       1423        489      19990          1          0

- **PRIORITY**: Final computed score — higher numbers mean higher placement in the queue.
- **AGE**: Time in queue (in minutes). Increases continuously until the job starts.
- **FAIRSHARE**: Normalized score based on your group's recent usage. Higher = better.
- **JOBSIZE**: Reflects node/core/memory request — used in backfilling calculations.

Checking Resource Usage & Billing
=================================

Jobs are charged based on core-hours, calculated as:

.. code-block:: text

   Core Usage = Billable Unit (Usually Cores) × Walltime (in hours)

Examples:

- 1-hour job using 16 cores = 16 core-hours
- 6-hour job using 4 cores = 24 core-hours

To inspect the actual billing and requested resources:

.. code-block:: bash

   [root@login01 ~]# sacct -j <jobid> --format=JobID,JobName%25,User,Partition,State,Elapsed,CPUTime,NCPUS,ReqTRES%60,AllocTRES%60
   
   JobID                          JobName      User  Partition      State    Elapsed    CPUTime      NCPUS                                                      ReqTRES                                                    AllocTRES
   ------------ ------------------------- --------- ---------- ---------- ---------- ---------- ---------- ------------------------------------------------------------ ------------------------------------------------------------
   111111         example-job-name-abbrev   user123   parallel    RUNNING   00:00:31   00:20:40         40                          billing=40,cpu=4,mem=160000M,node=1                         billing=40,cpu=40,mem=160000M,node=1
   111111.batch                     batch                         RUNNING   00:00:31   00:20:40         40                                                                                                 cpu=40,mem=160000M,node=1
   111111.0                        python                         RUNNING   00:00:22   00:14:40         40                                                                                                 cpu=40,mem=160000M,node=1

In this example:

- The user requested 40 CPUs and 160 GB of memory.
- `billing=40` confirms CPU is the dominant billing metric.
- `CPUTime` is the actual total core-time used so far: 

.. code-block:: console

    Cores = 40
    Elapsed Run Time = 00:00:31
    40 cores × 00:00:31 = 00:20:40 (CPUTime)

Common Pending Reasons
======================

When a job is in the **PENDING (PD)** state, Slurm includes a reason to help you understand **why it hasn’t started yet**. You can view this reason using:

.. code-block:: bash

   sqme

Below is a sample of output with example reasons:

.. code-block:: none

   JOBID            PARTITION  NAME      USER     ST  TIME      NODES  CPUS  REASON
   500001           parallel   sim01     user01   PD  0:00         1     1    (MaxCpuPerAccount)
   500002           parallel   sim02     user01   PD  0:00         1     1    (MaxCpuPerAccount)
   500003           parallel   jobXYZ    user02   PD  0:00         1     1    (AssocGrpCPUMinutesLimit)
   500004_[1-5]     parallel   arrayjob  user03   PD  0:00         1     1    (AssocGrpCPUMinutesLimit)
   500009           parallel   depend    user05   PD  0:00         1     1    (Dependency)


Common reasons include:

- **None**: The job hasn't had a reason assigned to it yet.
- **Priority**: Other jobs are ahead of yours in the queue.
- **Dependency**: Job is waiting on a prior job to complete (from a dependency chain).
- **Resources**: No nodes available that meet your job’s request (cores/memory/GPU).
- **JobArrayTaskLimit** – The job is part of an array and has reached its task concurrency limit.

CPU Related Reasons:

- **MaxCpuPerAccount**: The user or account has exceeded the limit on the job's QOS
- **AssocGrpCPUMinutesLimit**: The account or association has exceeded its allowed CPU time (core-minute) limit for the billing cycle. Jobs will stay pending until usage drops back below the threshold.

GPU Related Reasons: 

- **MaxGRESPerAccount/User**: Your group or user has reached a maximum GPU usage cap.
- **QOSMaxGRESPerUser** – The user has requested more GPUs than allowed for the current QoS.
- **MaxGRESPerAccount** – The account has hit a GRES (e.g., GPU) usage cap.

To see all potential job reason codes, refer to the official Slurm documentation:  
https://slurm.schedmd.com/job_reason_codes.html

Backfilling
===========

The scheduler attempts to maximize utilization by running small or short jobs in gaps between large reservations — this is called **backfilling**. This is why small jobs may sometimes appear to "jump the line" ahead of older, larger jobs. This helps improve efficiency without delaying higher-priority jobs.

Viewing Historical Usage and Efficiency
=======================================

For more on viewing efficiency and job usage statistics, refer to the page:

:doc:`Job_Status`

This includes guidance on using:

- `sacct` for historical usage
- `seff` and `reportseff` for job efficiency
- `jobstats` for GPU and memory metrics