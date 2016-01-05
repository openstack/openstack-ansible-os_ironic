Ironic Role for OpenStack-Ansible
#################################

This is a role for the deployment of Ironic in an `OpenStack-Ansible`_ environment.

Please see the `role-ironic spec`_ for more details.

.. _OpenStack-Ansible: https://github.com/openstack/openstack-ansible
.. _role-ironic spec: https://github.com/openstack/openstack-ansible-specs/blob/master/specs/mitaka/role-ironic.rst

Basic Role Example
^^^^^^^^^^^^^^^^^^

Tell us how to use the role.

.. code-block:: yaml

    - name: Playbook for role testing
      hosts: localhost
      remote_user: root
      roles:
        - role: openstack-ansible-ironic

