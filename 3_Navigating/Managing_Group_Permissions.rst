Manage Group Permissions
########################

Understanding and managing Unix group permissions is essential when working in a shared computing environment like Rockfish. Each file or directory has a user owner and a group owner, and permissions control how these resources can be accessed or modified.

This guide explains how group ownership and permissions work on Rockfish and how to manage them to support collaboration within research groups.

Overview
********

Unix permissions are important for:

- **Sharing data with other group members**
- **Maintaining correct group ownership for quota tracking**
- **Ensuring project members can read/write shared files**

Every user belongs to one or more Unix groups, typically based on project membership. When a user creates a file, the group ownership is usually set to their **current group**, which can be changed as needed.

Checking File Permissions and Group Ownership
**********************************************

To inspect ownership and permissions:

.. code-block:: console

   ls -l

Example output:

.. code-block:: console

   -rw-------  1 alice    projX     13511 Feb 14 10:00 data.txt
   -rwxr--r--  1 bob      projY       215 Sep 19 09:21 run.sh

- The third column shows the **owner** (user)
- The fourth column shows the **group owner**
- Permissions are shown in three sections:
  - **User (owner)**
  - **Group**
  - **Others**

If `alice` wants to share `data.txt` with other members of `projX`, she must ensure the group has the correct permissions:

.. code-block:: console

   chmod g+r data.txt

Changing Group Ownership
*************************

To change the group ownership of a file:

.. code-block:: console

   chgrp projX shared_results.txt

You can only change to groups you’re already a member of.

Viewing Your Group Membership
******************************

To list your current and supplementary groups:

.. code-block:: console

   groups

The first group listed is your **current (active)** group.

Setting the Active Group
*************************

To change your current group for a session:

.. code-block:: console

   newgrp projX

This is useful to ensure newly created files are assigned to the appropriate group.

You can also run a single command as a different group using:

.. code-block:: console

   sg projX <command>

Example:

.. code-block:: console

   sg projX mkdir new_folder

Default Group on Login
**********************

Your default group (first in the `groups` output) is used each time you log in. If this group is outdated or not associated with your current work, contact `help@rockfish.jhu.edu` to request a change.

Group Permissions and Quotas
*****************************

On Rockfish, group ownership determines not just access but also **quota tracking**. Storage usage in `/data/`, `/scratch4/`, and `/scratch16/` is accounted by group, and files owned by the wrong group may cause usage reports to be inaccurate.

To avoid issues:

- Always verify you're in the correct group before writing files
- Use `newgrp` if needed to switch context
- Use `quotas.py` to verify usage for the right group

Summary of Useful Commands
***************************

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - Command
     - Purpose
   * - `ls -l`
     - View file ownership and permissions
   * - `chmod g+r file`
     - Add read access for the group
   * - `chgrp groupname file`
     - Change file’s group owner
   * - `groups`
     - Show your group memberships
   * - `newgrp groupname`
     - Change your active group for the session
   * - `sg groupname <command>`
     - Run one command using a different group context