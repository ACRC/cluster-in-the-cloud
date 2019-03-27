Finalising the setup
====================

Terraform will have automatically started the cluster software configuration step.
It will run in the background and will take some time to complete.
In the meantime, you can connect to the custer and follow its progress.

Setting service limits
----------------------

You can log into the management node at ``yourusername@mgmtipaddress``,
using the IP address that terraform printed at the end of its run. You can `forward
your SSH key <https://developer.github.com/v3/guides/using-ssh-agent-forwarding/>`_
from your workstation to avoid copying the private key in to the cloud. First test 
that you can talk to the SSH agent by asking it to list the keys:

.. code-block:: shell-session

   $ ssh-add -L

Then connect to the managment node. For example:

.. code-block:: shell-session

   $ ssh -A opc@130.61.43.69

Once logged in, you can run the ``finish`` script:

.. code-block:: shell-session

   [opc@mgmt ~]$ ./finish

It will most likely tell you that the system has not finished configuring.
If the ``finish`` script is not there, wait a minute or two and it should appear.

To follow the progress, you can look at the file ``ansible-pull.log`` in ``root``'s home directory:

.. code-block:: shell-session

    [opc@mgmt ~]$ sudo tail -f /root/ansible-pull.log

Use ``ctrl-c`` to stop following the log.

You can keep on running trying to run ``finish`` until the node has finished configuring.
Once it has, you need to tell the system what limits you want to place on the scaling of the cloud.
To decide what to put here, you should refer to your service limits in the OCI website.
In the future, we hope to be able to extract these automatically but for now you need to replicate it manually.
Edit the file ``/home/opc/limits.yaml`` with:

.. code-block:: shell-session

   [opc@mgmt ~]$ vim limits.yaml

and set its contents to somthing like:

.. code-block:: yaml

   VM.Standard2.1:
     1: 1
     2: 1
     3: 1
   VM.Standard2.2:
     1: 1
     2: 1
     3: 1

which specifies for each shape, what the service limit is for each AD in the region your cluster lives in.
In this case each of the shapes ``VM.Standard2.1`` and ``VM.Standard2.2`` have a service limit of 1 in each AD.
The system will automatically adjust for the shape used by the management node.

Run ``finish`` again and it should configure and start the Slurm server:

.. code-block:: shell-session

   [opc@mgmt ~]$ ./finish

If your service limits change, you can update the file and run the script again.

Adding users
------------

To add users to the system, you run the command ``/usr/local/sbin/add_user_ldap`` passing it the username of the user you want to add,
the user's first and surnames and the URL of a file containing their SSH public keys.

.. code-block:: shell-session

   [opc@mgmt ~]$ sudo /usr/local/sbin/add_user_ldap matt Matt Williams https://github.com/milliams.keys

You can run this command again to add another user.

Once it has succeeded, log out and try logging as one of those users.

Check Slurm is running
----------------------

.. code-block:: shell-session

   $ ssh -A matt@130.61.43.69

Once logged in, try running the ``sinfo`` command to check that Slurm is running:

.. code-block:: shell-session

   [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      0    n/a

It shows the number of nodes being zero as the nodes will be automatically created as they are required,
up to the limit specified in the earlier step. This is all looking good so let's start submitting jobs.

Check out the information on :doc:`running the cluster <running>`.
