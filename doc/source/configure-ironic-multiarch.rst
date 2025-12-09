===============================================================
Deploying multiple Ironic nodes with different CPU architecures
===============================================================

Ironic can deploy nodes with CPU architectures which do not match
the CPU architecture of the control plane. The default settings for
OpenStack-Ansible assume an x86-64 control plane deploying x86-64
Ironic nodes.

This documentation describes how to deploy aarch64 Ironic nodes
using an x86-64 control plane. Other combinations of architecture
could be supported using the same approach with different variable
definitions.

This example assumes that Glance is used for Ironic image storage
and the Ironic control plane web server serves these for deployment.

Building ironic-python-agent deploy image for aarch64
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There must be an ironic-python-agent kernel and initramfs built and
uploaded to Glance for each architecture that needs to be deployed.

To build an aarch64 ironic-python-agent image using a Rocky Linux
aarch64 host:

.. code-block:: bash

  dnf install python3-virtualenv git qemu-img

  virtualenv venv
  source ./venv/bin/activate
  pip install ironic-python-agent-builder

  export DIB_REPOREF_ironic_python_agent=origin/master
  export DIB_REPOREF_requirements=origin/master
  ironic-python-agent-builder -o my-ipa --extra-args=--no-tmpfs --release 9-stream centos

- Replace ``origin/master`` with another branch reference in order to
  build specific versions of IPA, for example ``stable/zed``

Configuring Ironic for multiple architectures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This configuration assumes the use of iPXE. The settings required to
support an additional architecture are minimal. Ironic has a default setting
for which EFI firmware file to use that can be overridden on a per-architecture
basis with the ``ipxe_bootfile_name_by_arch`` config setting.

On the control plane the aarch64 EFI iPXE firmware must be present in
the tftp server root directory. Note that not all distributions supply
packages for EFI firmware for architectures different to the host so
it may be necessary to download architecture specific firmware directly
from https://boot.ipxe.org

This example shows how to specify the iPXE boot firmware to use for
aarch64 nodes, and where that firmware should be obtained from to
populate the tftp server.

.. code-block::

  ironic_ironic_conf_overrides:
    # Point to aarch64 uefi firmware on aarch64 platforms
    pxe:
      ipxe_bootfile_name_by_arch: 'aarch64:ipxe_aa64.efi'

  ironic_tftp_extra_content:
    - url: http://boot.ipxe.org/arm64-efi/ipxe.efi
      name: ipxe_aa64.efi

.. note::

  You must combine this override with any existing definition of
  ``ironic_ironic_conf_overrides``.

Enrolling an aarch64 node
~~~~~~~~~~~~~~~~~~~~~~~~~

When enrolling an aarch64 node the ``boot_mode`` must be uefi even
if existing Ironic nodes use legacy bios boot.

An example of the node capabilities including uefi boot would be:

.. code-block:: bash

  capabilities='boot_option:local,disk_label:gpt,boot_mode:uefi'

Enrolling an aarch64 node is exactly the same as enrolling an x86_64
node, except that ``deploy_kernel`` and ``deploy_ramdisk`` must be
set to the aarch64 version of the deploy image.

Building an aarch64 user image
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Example of building a whole-disk aarch64 user image on an existing
aarch64 Ubuntu host:

.. code-block:: bash

  sudo apt update
  sudo apt install python3-venv qemu-utils
  python3 -m venv venv
  source ./venv/bin/activate
  pip install diskimage-builder
  DIB_RELEASE=jammy DIB_CLOUD_INIT_DATASOURCES=Ec2 disk-image-create -a arm64 ubuntu vm block-device-efi cloud-init-datasources -o baremetal-ubuntu-22.04-efi-arm64.qcow2

- The DIB_RELEASE=<name> environment variable tells the 'ubuntu'
  element which version of Ubuntu to create an image for. This defaults
  to Focal if left unspecified.

- The DIB_CLOUD_INIT_DATASOURCES=Ec2 environment variable is used
  by the ``cloud-init-datasources`` element to force ``cloud-init`` to use
  its Ec2 datasource. The native OpenStack datasource can't be used
  because it doesn't currently have working support for bare metal
  instances until ``cloud-init`` version 23.1. (Since the OpenStack
  metadata service also provides an EC2 compatible API, the Ec2 datasource
  is a reasonable workaround. (NB: This is actually the default behaviour
  for Ubuntu cloud images, but for entirely unrelated reasons hence it
  being worth making explicit here.)

Use a similar approach on a Rocky Linux aarch64 system to build
a whole-disk user image of the latest version of Rocky Linux:

.. code-block:: bash

  DIB_RELEASE=9 DIB_CLOUD_INIT_DATASOURCES=Ec2 DIB_CLOUD_INIT_GROWPART_DEVICES='["/"]' disk-image-create -a arm64 rocky-container vm block-device-efi cloud-init openssh-server cloud-init-datasources cloud-init-growpart -o baremetal-rocky-9-efi-arm64.qcow2

- The DIB_RELEASE=<number> environment variable tells the 'rocky-container'
  element which version of Rocky to create an image for.
- The ``cloud-init`` and ``openssh-server`` elements are essential since the
  Rocky container image does not include these packages. (As an aside:
  the ``diskimage-builder`` documentation erroneously claims that the
  ``cloud-init`` element only works on Gentoo, but this is not the case).
- As with Ubuntu, setting DIB_CLOUD_INIT_DATASOURCES=Ec2 and using the
  ``cloud-init-datasources`` element is necessary since the OpenStack
  ``cloud-init`` datasource doesn't work. Unlike the Ubuntu case, using the
  Ec2 datasource is not the default and so adding these options is
  essential to obtain a working image.
- DIB_CLOUD_INIT_GROWPART_DEVICES variable tells cloud-init-growpart to
  configure cloud-init to grow the root partition on first boot which is
  must be explicitly set on some OS/architecture combinations.
