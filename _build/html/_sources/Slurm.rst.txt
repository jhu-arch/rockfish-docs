Job Scheduling with Slurm
#########################

Rockfish uses **Slurm (Simple Linux Universal Resource Manager)** to manage job submission, scheduling, and resource allocation. It is a widely adopted, open-source workload manager used by many HPC centers.

All jobs must be submitted through Slurm — running compute-heavy processes directly on login nodes is not permitted.

.. contents::
   :local:
   :depth: 2

Overview of Job Types
*********************

- **Batch Jobs**: Standard non-interactive jobs submitted with `sbatch`.
- **Interactive Jobs**: Allocate compute resources for a live terminal session using `interact`.
- **Job Arrays**: Submit a group of related jobs with a single script using `--array`.

Available Partitions
*********************

Slurm divides resources into **partitions** (similar to queues). Each partition targets specific hardware or workloads.

.. list-table::
   :header-rows: 1
   :widths: 20 15 15 15 20

   * - Partition
     - Max Time (hrs)
     - Cores per Node
     - Max Memory
     - Use Case
   * - parallel
     - 72
     - 48
     - 192 GB
     - General use
   * - bigmem
     - 48
     - 48
     - 1.5 TB
     - Large-memory jobs
   * - a100
     - 72
     - 48
     - 192 GB
     - GPU jobs (A100)
   * - ica100
     - 72
     - 64
     - 256 GB
     - GPU jobs
   * - shared
     - 24
     - 64
     - 256 GB
     - Shared cores
   * - express
     - 8
     - 128
     - 256 GB
     - Short runs

Basic Job Submission
*********************

Submit a batch script:

.. code-block:: console

   sbatch my_script.sh

Cancel a job:

.. code-block:: console

   scancel <job_id>

Display your jobs:

.. code-block:: console

   sqme

Example Batch Script
=====================

.. code-block:: bash

   #!/bin/bash
   #SBATCH --job-name=MyJob
   #SBATCH --time=24:00:00
   #SBATCH --partition=shared
   #SBATCH --nodes=1
   #SBATCH --ntasks-per-node=24
   #SBATCH --mail-type=end
   #SBATCH --mail-user=userid@jhu.edu

   module load intel/2020.2 intel-mpi/2020.2
   mpirun -n $SLURM_NTASKS ./my_program.x > output.log

Interactive Jobs
****************

Use `interact` to request a live session on compute nodes:

.. code-block:: console

   interact -p parallel -n 4 -t 01:00:00

GPU interactive job:

.. code-block:: console

   interact -p a100 -g 1 -n 6 -t 02:00:00

See all interact options:

.. code-block:: console

   interact --usage

Slurm Environment Variables
****************************

.. list-table::
   :header-rows: 1
   :widths: 30 40

   * - Variable
     - Description
   * - `$SLURM_JOBID`
     - Unique ID of the current job
   * - `$SLURM_JOB_NODELIST`
     - Nodes assigned to the job
   * - `$SLURM_ARRAY_TASK_ID`
     - Task index for array jobs
   * - `$SLURM_CPUS_PER_TASK`
     - Cores per task
   * - `$SLURM_SUBMIT_DIR`
     - Directory where job was submitted

Requesting Resources
*********************

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Option
     - Description
   * - `--nodes=1`
     - Request 1 node
   * - `--ntasks=24`
     - Total MPI processes (tasks)
   * - `--cpus-per-task=6`
     - Threads per task
   * - `--mem=120GB`
     - Memory per node
   * - `--mem-per-cpu=4GB`
     - Memory per CPU
   * - `--exclusive`
     - Use the entire node
   * - `--shared`
     - Share the node with others
   * - `--account=myaccount`
     - Specify a Slurm account
   * - `--qos=qos_gpu`
     - Set quality-of-service

GPU Jobs
********

To run on GPU nodes:

.. code-block:: bash

   #SBATCH -p a100
   #SBATCH --gres=gpu:2
   #SBATCH --ntasks-per-node=2
   #SBATCH --cpus-per-task=6

You can check which GPUs were assigned with:

.. code-block:: bash

   echo $CUDA_VISIBLE_DEVICES

Job Arrays
**********

Useful for submitting many similar jobs at once:

.. code-block:: console

   sbatch --array=0-15%4 script.sh

- Runs jobs with IDs 0 to 15
- `%4` limits to 4 concurrent jobs

To name array logs dynamically:

.. code-block:: bash

   #SBATCH -o slurm-%A_%a.out

Where `%A` is the job array ID and `%a` is the task ID.

Output File Routing
********************

Customize standard output and error paths:

.. code-block:: bash

   #SBATCH -o /home/userid/logs/%j_%x.out
   #SBATCH -e /home/userid/logs/%j_%x.err

Where:

- `%j`: Job ID
- `%x`: Job name

Viewing Job Status & Efficiency
*******************************

- `squeue`: See current jobs in queue
- `scontrol show job <id>`: View full job details
- `seff <id>`: Show job efficiency
- `jobstats <id>`: Get GPU usage statistics

Useful Commands Summary
***********************

.. list-table::
   :header-rows: 1
   :widths: 35 65

   * - Command
     - Description
   * - `sbatch script.sh`
     - Submit batch job
   * - `squeue -u $USER`
     - List your jobs
   * - `scancel <job_id>`
     - Cancel a job
   * - `scontrol show job <job_id>`
     - Detailed job info
   * - `seff <job_id>`
     - Efficiency report
   * - `interact`
     - Start interactive session
   * - `sacct`
     - View historical jobs
   * - `sqme`
     - Show user jobs (custom shortcut)