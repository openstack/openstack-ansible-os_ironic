===================================================
Debugging the Bare Metal (ironic) inspector service
===================================================

Ironic Python Agent debug logs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If IPA fails, a log file will be written to ``/var/log/ironic``
on the conductor node responsible for the Ironic node being deployed.

A lot of useful information is collected including a copy of the journal
log from the host.

Pausing during a deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To pause the deployment, possibly to log into an Ironic node running the
IPA to do diagnostic work, the node can have ``maintainance``
switched on either through the OpenStack CLI or Horizon web interface.

The ironic state machine will not change state again until maintainance mode
is switched off, triggered by a heartbeat from IPA. Note
that you will have to wait for up to one heartbeat period for the deployment
to resume after switching off maintainance mode.

Logging into IPA
~~~~~~~~~~~~~~~~

To perform diagnostic steps on an ironic node during deployment or cleaning
it might be helpful to be able to log into the node from the controller after
the deploy image has booted.

To build a version of the deploy image which has the ``dynamic-login`` element
enabled, in this case building on a Rocky Linux host:

.. code-block:: bash

  dnf install python3-virtualenv git qemu-img

  virtualenv venv
  source ./venv/bin/activate
  pip install ironic-python-agent-builder

  export DIB_REPOREF_ironic_python_agent=origin/stable/zed
  export DIB_REPOREF_requirements=origin/stable/zed
  ironic-python-agent-builder -e dynamic-login -o my-login-ipa --extra-args=--no-tmpfs --release 8-stream centos

Once the IPA kernel and initramfs are built, upload them to glance and
set them as the deploy kernel/initramfs for the Ironic node to log into
during deployment.

Create a password to log into the Ironic node:

.. code-block:: bash

  openssl passwd -1 -stdin <<< YOUR_PASSWORD | sed 's/\$/\$\$/g'

Set debugging options on the IPA kernel parameters
in ``/etc/openstack_deploy/user_variables.yml``, substituing the encrypted
password just generated into the ``rootpwd`` field. Ensure that the
encrypted password is enclosed in double quotation marks.

.. code-block:: bash

  ironic_ironic_conf_overrides:
    # Set a password on the root user in IPA for debug purposes
    pxe:
      kernel_append_params: 'ipa-debug=1 systemd.journald.forward_to_console=yes rootpwd="<password-string>"'

.. note::

  You must combine this override with any existing definition of
  ``ironic_ironic_conf_overrides``.

Deploy the configuration file changes to the Ironic control plane.

The node can now be cleaned or provisioned, possibly pausing the deployment
by enabling maintainance on the node and then logging into the node
with SSH as the root user with the password previously encrypted.
