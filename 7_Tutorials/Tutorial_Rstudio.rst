RStudio Server
##############

The open-source RStudio Server provides a fully-featured IDE for R users. You can access **RStudio Server** on Rockfish in two different ways:

Access via Open OnDemand (OOD)
==============================

1. **Connect to the Hopkins VPN**  
   Access to OOD is restricted to the JHU network.

2. **Open the OOD Portal**  
   Go to: https://portal.rockfish.jhu.edu

3. **Login**  
   Use your Rockfish username and password.

4. **Launch RStudio Server**  
   On the main dashboard, click the ``RStudio Server`` icon.

5. **Configure Job Parameters:**
   - **R Version:** Choose the desired R version (e.g., ``R/4.4.1 - RStudio Server``)
   - **R_LIBS_USER:** Specify the path to your personal R library
   - **Wall Time:** Total time you plan to use RStudio
   - **Number of Cores:** Each core provides ~4 GB RAM — choose based on memory needs
   - **Partition:** Select based on resource needs (see details in OOD)

6. **Launch Session**  
   Click **Launch** and wait for the session to initialize. Once ready, click the blue **Connect to RStudio Server** button.

Access via ``r-studio-server.sh`` Script
========================================

Use this method if you prefer more control over your session or are scripting workflows.

Generate SLURM Script
---------------------

Run the helper script to create your session job:

Usage examples:

.. code-block:: bash

   $ r-studio-server.sh -h
   $ r-studio-server.sh -n 1 -c 2 -m 8G -t 1-02:0 -p defq
   $ r-studio-server.sh -c 2 -t 4:0:0 -p defq -e <userid>@jhu.edu
   $ r-studio-server.sh -c 24 -g 2 -p a100 -a <PI-userid>_gpu

Parameter Guide:
~~~~~~~~~~~~~~~~
- ``-c``: Number of cores (~4 GB RAM per core)
- ``-t``: Runtime in HH:MM:SS
- ``-p``: Partition

  - express: up to 4 cores / 8 hours  
  - shared: up to 32 cores / 36 hours  
  - parallel: up to 48 cores / 72 hours

You can run ``r-studio-server.sh --help`` to see all available options.

Edit the Generated Files
------------------------

The script will generate **two files**:

a) ``.r-studio-variables`` (hidden config file)

.. code-block:: console

   Creating slurm script: R-Studio-Server.slurm.script
   The Advanced Research Computing at Hopkins (ARCH)
   SLURM job script for RStudio in Singularity container
   Support: help@rockfish.jhu.edu

   Nodes:         2
   Cores/task:    4
   Total cores:   8
   Walltime:      00-02:00
   Queue:         defq

   The R-Studio-Server is ready to run.

   1 - Usage:
     $ sbatch R-Studio-Server.slurm.script

   2 - How to login (after step 1):
     $ cat rstudio-server.job.<SLURM_JOB_ID>.out

   3 - More job info:
     $ scontrol show jobid <SLURM_JOB_ID>

.. tip::
   The ``#SBATCH`` tags can be customized. Also, the R environment can be changed to use another version instead of the container default (R 4.0.4).

.. warning::
   Make sure the ``anaconda`` module is not loaded when you submit the job.

   Unload it with:

   .. code-block:: bash

      module unload anaconda

Example ``R-Studio-Server.slurm.script``:
-----------------------------------------

.. code-block:: shell

   #!/bin/bash
   #####################################
   #SBATCH --job-name=rstudio_container_rdesouz4
   #SBATCH --time=1-02:0
   #SBATCH --partition=defq
   #SBATCH --signal=USR2
   #SBATCH --nodes=1
   #SBATCH --cpus-per-task=2
   #SBATCH --mem=8G
   #SBATCH --mail-type=END,FAIL
   #SBATCH --mail-user=rdesouz4@jh.edu
   #SBATCH --output=rstudio-server.job.%j.out
   #####################################

   # R environment options (choose one):

   # METHOD 1: Use an R via system module
   # module load r/3.6.3

   # METHOD 2: Use R via conda environment
   # module load anaconda && conda activate r_3.6.3 && export VIRT_ENV=$CONDA_PREFIX && module unload anaconda
   # export R_HOME=${VIRT_ENV}/lib/R
   # export SINGULARITY_BIND=${R_HOME}:/usr/local/lib/R

   export MY_LIBS=4.0.4
   export R_LIBS_USER=${HOME}/R/${MY_LIBS}

   source .r-studio-variables

   export SINGULARITYENV_LDAP_HOST=ldapserver
   export SINGULARITYENV_LDAP_USER_DN='uid=%s,dc=cm,dc=cluster'
   export SINGULARITYENV_LDAP_CERT_FILE=/etc/rstudio/ca.pem

   cat 1>&2 <<END

   1. SSH tunnel from your workstation:
      ssh -N -L \${PORT}:\${HOSTNAME}:\${PORT} \${SINGULARITYENV_USER}@login.rockfish.jhu.edu

   2. Log in to RStudio in your browser:
      http://localhost:\${PORT}

      user: \${SINGULARITYENV_USER}
      password: <Rockfish password>

   3. To end the session:
      a. Click the "power" button in RStudio
      b. Run:
         scancel -f \${SLURM_JOB_ID}

   END

   singularity run \${SINGULARITY_CONTAINER} \
   rserver --www-port \${PORT} --www-address=0.0.0.0 \
           --auth-none 0 \
           --auth-pam-helper-path=ldap_auth \
           --rsession-path=/etc/rstudio/rsession.sh

Customizing R Version
---------------------

Edit ``.r-studio-variables``:

.. code-block:: bash

   nano .r-studio-variables

Replace line 13:

.. code-block:: bash

   module load gfbf/2023b RStudio-Server/2023.12.1+402-gfbf-2023b-Java-11-R-4.4.1

Replace line 14:

.. code-block:: bash

   ml -R/4.4.1-gfbf-2023b

Save and exit (Ctrl + O, Enter; Ctrl + X)

Edit ``R-Studio-Server.slurm.script``:

.. code-block:: bash

   nano R-Studio-Server.slurm.script

After this line:

.. code-block:: bash

   module restore

Add:

.. code-block:: bash

   module load gfbf/2023b R/4.4.1-gfbf-2023b

Then locate:

.. code-block:: bash

   export R_LIBS_USER=${HOME}/R/4.2.1

Replace with:

.. code-block:: bash

   export R_LIBS_USER=/home/YourUserID/rlibs/R-4.4.1-gfbf-2023b

Make sure the directory exists.

Submit the Job
--------------

.. code-block:: bash

   sbatch R-Studio-Server.slurm.script

Output File
-----------

.. code-block:: bash

   ls -ltr rstudio-server.job*

View Connection Info
--------------------

.. code-block:: bash

   cat rstudio-server.job.<jobid>.out

Set Up SSH Tunnel (Local Machine)
---------------------------------

.. code-block:: bash

   ssh -N -L <PORT>:<node>:<PORT> YourUserID@login.rockfish.jhu.edu

Leave this terminal open.

Access RStudio in Browser
-------------------------

Locate this URL in your output file:

.. code-block:: text

   http://localhost:<PORT>

Log in with your Rockfish credentials.