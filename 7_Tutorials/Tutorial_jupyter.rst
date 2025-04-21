Jupyter
##################################

Accessing Jupyter Notebook on Rockfish
**************************************

You can access Jupyter Notebook on Rockfish in **two different ways**:

Option 1: Using the Web Portal (Open OnDemand)
**********************************************

1. Connect to the Hopkins VPN  
   Access to Open OnDemand (OOD) is only available from the JHU network.

2. Open your browser and go to:  
   https://portal.rockfish.jhu.edu

3. Login using your Rockfish username and password.

4. On the dashboard, click the **Jupyter Server** icon.

5. Choose your:
   - Wall Time
   - Number of Cores
   - Partition  
   Then click **Launch**.

6. A new page will open. When the Jupyter server is ready, click the **Connect to Jupyter** button.

Option 2: Using the ``jupyterlab.sh`` Script
*********************************************

For users who prefer working through the terminal, follow these steps:

Connect to Rockfish
===================

Open a terminal and run:

.. code-block:: bash

   ssh YourUserID@login.rockfish.jhu.edu

You’ll be prompted for your password — type it and press Enter.

Generate the Slurm Script
=========================

Run the following command with the parameters that fit your needs:

.. code-block:: bash

   jupyterlab.sh -n 1 -c 1 -t 01:00:00 -p a100 -q qos_gpu -g 1

This will create a file named:

.. code-block:: text

   jupyter_lab.slurm.script

Parameter notes:

- ``-c``: Number of cores (each core provides ~4 GB RAM)
- ``-t``: Job time (in HH:MM:SS)
- ``-p``: Partition  
  - ``express``: up to 4 cores / 8 hours  
  - ``shared``: up to 32 cores / 36 hours  
  - ``parallel``: up to 48 cores / 72 hours  
- ``-q``: (optional) QoS
- ``-g``: Number of GPUs (if needed)

Submit the Job
==============

.. code-block:: bash

   sbatch jupyter_lab.slurm.script

A file will be created named:

.. code-block:: text

   Jupyter_lab.job.<jobid>.login

Replace ``<jobid>`` with your actual SLURM job ID.

Retrieve SSH Tunnel Instructions
================================

List the output file:

.. code-block:: bash

   ls -ltr Jupyter_lab.job*

View its contents:

.. code-block:: bash

   cat Jupyter_lab.job.<jobid>.login

Set Up SSH Tunnel (Local Machine)
=================================

On your **local terminal** (not logged into Rockfish), copy and run the ``ssh -N -L ...`` command shown in the ``.login`` file.

You’ll be prompted for your Rockfish password — enter it and **minimize** the terminal (do not close it).

Access Jupyter in Your Browser
==============================

Back in the ``.login`` file, find the line starting with:

.. code-block:: text

   http://localhost:...

Copy and paste that URL into your browser.  
Log in using your Rockfish username and password — you're in!