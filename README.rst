Ironic Role for OpenStack-Ansible
#################################

This is a role for the deployment of Ironic in an `OpenStack-Ansible`_ environment.

Please see the `role-ironic spec`_ for more details.

.. _OpenStack-Ansible: https://github.com/openstack/openstack-ansible
.. _role-ironic spec: https://github.com/openstack/openstack-ansible-specs/blob/master/specs/mitaka/role-ironic.rst

Tags
====

This role supports two tags: ``ironic-install`` and ``ironic-config``

The ``ironic-install`` tag can be used to install and upgrade.

The ``ironic-config`` tag can be used to maintain configuration of the
service.
