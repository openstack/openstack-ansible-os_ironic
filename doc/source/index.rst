======================================================
OpenStack-Ansible role for Bare Metal (ironic) service
======================================================

.. toctree::
   :maxdepth: 2

   configure-ironic.rst

This is an OpenStack-Ansible role to deploy the Bare Metal (ironic)
service. See the `role-ironic spec`_ for more information.

.. _role-ironic spec: http://specs.openstack.org/openstack/openstack-ansible-specs/specs/mitaka/role-ironic.html


To clone or view the source code for this repository, visit the role repository
for `os_ironic <https://github.com/openstack/openstack-ansible-os_ironic>`_.

Default variables
~~~~~~~~~~~~~~~~~

.. literalinclude:: ../../defaults/main.yml
   :language: yaml
   :start-after: under the License.


Required variables
~~~~~~~~~~~~~~~~~~

None.


Example playbook
~~~~~~~~~~~~~~~~

.. literalinclude:: ../../examples/playbook.yml
   :language: yaml


Tags
====

This role supports the ``ironic-install`` and ``ironic-config`` tags.
Use the ``ironic-install`` tag to install and upgrade. Use the
``ironic-config`` tag to maintain configuration of the service.
