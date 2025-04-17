Viewing Job Status & Efficiency
###############################

- `sqme`: View currently running or queued jobs.
- `scontrol show job <id>`: View detailed information about a specific job.
- `sacct`: View job history and status.
- `seff <id>`: Check job efficiency (CPU/memory usage) once the job has run.
- `reportseff <id>`: Get a summary of job efficiency.
- `jobstats <id>`: See GPU usage statistics for active or completed jobs.

**Example: View All Jobs for a User with `sqme` (custom wrapper for `squeue`)**

.. code-block:: console

  [user@login02 ~]$ sqme
  USER   ACCOUNT      JOBID   PARTITION  NAME       NODES  CPUS  MIN_MEMORY  TIME_LIMIT  TIME     NODELIST  ST  REASON
  user   group_gpu    111111  a100       job1.sh    1      12    4000M       3-00:00:00  3:53:46  gpu14     R   None
  user   group_gpu    111112  a100       job2.sh    1      12    4000M       3-00:00:00  3:09:00  gpu13     R   None

**Example: Detailed Job Info with `scontrol show job`**

.. code-block:: console

  [user@login02 ~]# scontrol show job 1111111
  JobId=1111111 JobName=job_script.sh
     UserId=example_user GroupId=example_group MCS_label=N/A
     Priority=20688 Nice=0 Account=example_gpu QOS=qos_gpu
     JobState=RUNNING Reason=None Dependency=(null)
     Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
     RunTime=03:55:39 TimeLimit=3-00:00:00 TimeMin=N/A
     SubmitTime=2025-04-17T10:54:51 EligibleTime=2025-04-17T10:54:51
     StartTime=2025-04-17T10:55:00 EndTime=2025-04-20T10:55:00 Deadline=N/A
     Partition=a100 AllocNode:Sid=login01:2094294
     NodeList=gpu14
     NumNodes=1 NumCPUs=12 NumTasks=1 CPUs/Task=1
     ReqTRES=cpu=1,mem=4000M,node=1,billing=12,gres/gpu=1
     AllocTRES=cpu=12,mem=48000M,node=1,billing=12,gres/gpu=1,gres/gpu:a100=1
     Command=/scratch4/example_group/example_user/projects/example_project/job_script.sh
     StdErr=/scratch4/example_group/example_user/projects/example_project/logs/output.txt
     StdOut=/scratch4/example_group/example_user/projects/example_project/logs/output.txt

**Example: Job History with `sacct`**

.. code-block:: console

  [user@login02 ~]# sacct
  JobID           JobName    Partition    Account        AllocCPUS   State       ExitCode
  --------------  ---------  -----------  -------------  ----------  ----------  --------
  111111          job1.sh    a100         example_gpu          12    TIMEOUT     0:0
  111111.batch    batch      a100         example_gpu          12    CANCELLED   0:15
  111111.0        python     a100         example_gpu          12    COMPLETED   0:0
  111112          job2.sh    a100         example_gpu          12    RUNNING     0:0
  111112.batch    batch      a100         example_gpu          12    RUNNING     0:0
  111112.0        python     a100         example_gpu          12    RUNNING     0:0
  111113          job3.sh    a100         example_gpu          12    FAILED      1:0
  111113.batch    batch      a100         example_gpu          12    FAILED      1:0
  111114          job4.sh    a100         example_gpu          12    FAILED      1:0
  111114.batch    batch      a100         example_gpu          12    FAILED      1:0
  111115          job5.sh    a100         example_gpu          12    CANCELLED   0:0
  111115.batch    batch      a100         example_gpu          12    CANCELLED   0:15
  111116          job6.sh    a100         example_gpu          12    RUNNING     0:0
  111116.batch    batch      a100         example_gpu          12    RUNNING     0:0

**Example: Job Efficiency with `seff`**

.. code-block:: console

  [root@login02 ~]# seff 111111
  Job ID: 111111
  Cluster: slurm
  User/Group: example_user/example_group
  State: RUNNING
  Nodes: 1
  Cores per node: 12
  CPU Utilized: 00:00:00
  CPU Efficiency: 0.00% of 1-23:31:00 core-walltime
  Job Wall-clock time: 03:57:35
  Memory Utilized: 0.00 MB (estimated maximum)
  Memory Efficiency: 0.00% of 46.88 GB (3.91 GB/core)
  WARNING: Efficiency statistics may be misleading for RUNNING jobs.

**Example: Summary View of Efficiency with `reportseff`**

.. code-block:: console

  [root@login02 ~]# reportseff 111111
     JobID   State      Elapsed  TimeEff   CPUEff   MemEff
    111111  RUNNING    03:57:40   5.5%      ---      ---

**Example: GPU Usage Summary with `jobstats`**

**Note:**  
We use `jobstats`, an open-source utility developed by `Princeton University <https://github.com/PrincetonUniversity/jobstats>`__, to collect and visualize CPU, memory, and GPU utilization for Slurm jobs. It provides an intuitive, at-a-glance summary of resource efficiency and is particularly helpful for GPU workflows.


.. code-block:: console

  [user@login02 ~]# jobstats 1111111

  ================================================================================
                                Slurm Job Statistics
  ================================================================================
           Job ID: 1111111
    NetID/Account: example_user/example_group_gpu
         Job Name: job_script
            State: RUNNING
            Nodes: 1
        CPU Cores: 12
       CPU Memory: 4GB (333.3MB per CPU-core)
             GPUs: 1
    QOS/Partition: qos_gpu/a100
          Cluster: slurm
       Start Time: Wed Apr 16, 2025 at 4:59 PM
         Run Time: 21:43:28 (in progress)
       Time Limit: 1-00:00:00

                                Overall Utilization
  ================================================================================
    CPU utilization  [|||||                                          10%]
    CPU memory usage [||||||||                                       16%]
    GPU utilization  [|||||||||||||||||||||||||||||||||||||||||||||| 93%]
    GPU memory usage [|||||||||||||||                                31%]

                                Detailed Utilization
  ================================================================================
    CPU utilization per node (CPU time used/run time)
        gpu06: 1-01:34:12/10-20:41:43 (efficiency=9.8%)

    CPU memory usage per node - used/allocated
        gpu06: 7.4GB/46.9GB (631.9MB/3.9GB per core of 12)

    GPU utilization per node
        gpu06 (GPU 1): 92.7%

    GPU memory usage per node - maximum used/total
        gpu06 (GPU 1): 12.4GB/40.0GB (31.1%)

                                       Notes
  ================================================================================
    * Have a nice day!