================================================================
Configuring the Bare Metal (ironic) inspector service (optional)
================================================================

.. note::

   This feature is experimental at this time and it has not been fully
   production tested yet.

Ironic Inspector is an Ironic service that deploys a tiny image called
ironic-python-agent that gathers information about a Bare Metal node. The data
is then stored in the database for further use later. The node is then updated
with properties based in the introspection data.

The inspector configuration requires some pre-deployment steps to allow the
Ironic playbook to make the inspector functioning.

Networking
~~~~~~~~~~
Ironic networking must be configured as normally done. The inspector and
Ironic will both share the TFTP server.

Networking will depend heavily on your environment. For example, the DHCP for
both Ironic and inspector will come from the same subnet and will be a subset
of the typical ironic allocated range.


Required Overrides
~~~~~~~~~~~~~~~~~~
  .. code-block::

     # dnsmasq/dhcp information for inspector
     ironic_inspector_dhcp_pool_range: <START> <END> (subset of ironic IPs)
     ironic_inspector_dhcp_subnet: <IRONIC SUBNET CIDR>
     ironic_inspector_dhcp_subnet_mask: 255.255.252.0
     ironic_inspector_dhcp_gateway: <IRONIC GATEWAY>
     ironic_inspector_dhcp_nameservers: 8.8.8.8

To enable LLDP discovery of switch ports during inspection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During inspection Ironic Inspector can automatically populate
information into the node ``local_link_connection`` which can
automatically create a baremetal port for the node.

This example is suitable for switches that have a common MAC address
per switch port and are identified to networking-generic-switch
using the ``ngs_mac_address`` parameter which matches against
the ``switch_id`` field in the Ironic node ``local_link_connection``
information.

Set the following variables in ``/etc/openstack_deploy/user_variables.yml``

.. code-block:: yaml

  # enable LLDP discovery for inspector
  ironic_inspector_processing_hooks: "$default_processing_hooks,lldp_basic,local_link_connection"
  ironic_inspector_extra_callback_parameters: "ipa-collect-lldp=1"

To enable LLDP discovery of switch system name during inspection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example is suitable for switches that have a different MAC address
per switch port and are identified to networking-generic-switch
using the switch hostname which is matched against
the ``switch_info`` field in the Ironic node ``local_link_connection``
information.

An additional out-of-tree Ironic Inspector plugin is needed to
obtain the system name of the switch and write it to ``switch_info``
during inspection.

Set the following variables in ``/etc/openstack_deploy/user_variables.yml``

.. code-block:: yaml

  # enable LLDP discovery for inspector
  ironic_inspector_processing_hooks: "$default_processing_hooks,lldp_basic,local_link_connection,system_name_llc"
  ironic_inspector_extra_callback_parameters: "ipa-collect-lldp=1"

  # stackhpc inspector plugins
  ironic_inspector_user_pip_packages:
    - git+https://github.com/stackhpc/stackhpc-inspector-plugins@master#egg=stackhpc-inspector-plugins
