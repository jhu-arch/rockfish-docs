Frequently Asked Questions
##########################

.. contents::
   :local:
   :depth: 1

.. dropdown:: What type of data can I upload to Rockfish?

   Data subject to restrictions such as HIPAA or PHI is **not permitted** on Rockfish.  
   If your research involves an IRB and the data is de-identified, please reach out to  
   `help@rockfish.jhu.edu <mailto:help@rockfish.jhu.edu>`__ for further guidance.

.. dropdown:: How do I connect to Rockfish?

   Use your JHED ID and run:

   .. code-block:: console

      ssh -XY <your_jhed>@login.rockfish.jhu.edu

.. dropdown:: What default resources do I receive?

   - 50GB `/home/` directory
   - 10TB group allocation on `/data/`
   - 1TB group allocation on `/scratch4/`
   - /scratch16/ access available upon request

.. dropdown:: How do I request an allocation?

   PIs must submit a short proposal through the Coldfront Portal.  
   Allocations are available for standard, GPU, and large-memory usage.  
   Startup allocations are also available for benchmarking.

.. dropdown:: How do I find my group or personal utilization?

   .. code-block:: console

      sbalance -a <group>
      test-sbalance -u $USER

.. dropdown:: How do I request interactive jobs?

   Use the `interact` utility to request interactive sessions.

   .. code-block:: console

      interact -usage

.. dropdown:: How do I select the correct Slurm account?

   If you have access to multiple accounts, specify one using:

   .. code-block:: console

      #SBATCH -A johndoe1

.. dropdown:: How do I attach to a compute node where my job is running?

   First, find the job and node using `sqme`, then attach:

   .. code-block:: console

      srun --jobid=<job_id> -w <node> --pty /bin/bash

.. dropdown:: How do I check job efficiency?

   Use `seff`, `reportseff`, or `jobstats` after your job completes:

   .. code-block:: console

      seff <job_id>
      reportseff <job_id>
      jobstats <job_id>

.. dropdown:: How do I transfer large datasets?

   Use Globus with the **Rockfish User Data** endpoint.  
   For large numbers of small files, compress them into tarballs first:

   .. code-block:: console

      tar -czf mydata.tgz mydata/

.. dropdown:: How do I use FileZilla?

   - Host: `rfdtn1.rockfish.jhu.edu`
   - Port: `22`
   - Protocol: `SFTP – SSH File Transfer Protocol`
   - Login Type: `Interactive`
   - Limit simultaneous transfers to **1** in Transfer Settings

   Your Rockfish username should be used for login (e.g., `jdoe1234`).