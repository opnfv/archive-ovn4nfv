===============
OVN information
===============

The original OVN project announcement can be found here:

* http://networkheresy.com/2015/01/13/ovn-bringing-native-virtual-networking-to-ovs/

The OVN architecture is described here:

* http://openvswitch.org/support/dist-docs/ovn-architecture.7.html

Here are two tutorials that help with learning different aspects of OVN:

* http://blog.spinhirne.com/p/blog-series.html#introToOVN
* http://docs.openvswitch.org/en/latest/tutorials/ovn-sandbox/

There is also an in depth tutorial on using OVN with OpenStack:

* http://docs.openvswitch.org/en/latest/tutorials/ovn-openstack/

OVN DB schemas and other man pages:

* http://openvswitch.org/support/dist-docs/ovn-nb.5.html
* http://openvswitch.org/support/dist-docs/ovn-sb.5.html
* http://openvswitch.org/support/dist-docs/ovn-nbctl.8.html
* http://openvswitch.org/support/dist-docs/ovn-sbctl.8.html
* http://openvswitch.org/support/dist-docs/ovn-northd.8.html
* http://openvswitch.org/support/dist-docs/ovn-controller.8.html
* http://openvswitch.org/support/dist-docs/ovn-controller-vtep.8.html

or find a full list of OVS and OVN man pages here:

* http://docs.openvswitch.org/en/latest/ref/

The openvswitch web page includes a list of presentations, some of which are
about OVN:

* http://openvswitch.org/support/

Here are some direct links to past OVN presentations:

* `OVN talk at OpenStack Summit in Boston, Spring 2017
  <https://www.youtube.com/watch?v=sgc7myiX6ts>`_
* `OVN talk at OpenStack Summit in Barcelona, Fall 2016
  <https://www.youtube.com/watch?v=q3cJ6ezPnCU>`_
* `OVN talk at OpenStack Summit in Austin, Spring 2016
  <https://www.youtube.com/watch?v=okralc7LrZo>`_
* OVN Project Update at the OpenStack Summit in Tokyo, Fall 2015 -
  `Slides <http://openvswitch.org/support/slides/OVN_Tokyo.pdf>`__ -
  `Video <https://www.youtube.com/watch?v=3IrG2xghJjs>`__
* OVN at OpenStack Summit in Vancouver, Sping 2015 -
  `Slides <http://openvswitch.org/support/slides/OVN-Vancouver.pdf>`__ -
  `Video <https://www.youtube.com/watch?v=kEzXTq2fPDg>`__
* `OVS Conference 2015 <https://www.youtube.com/watch?v=JLGZOYi_Cqc>`_

These blog resources may also help with testing and understanding OVN:

* http://networkop.co.uk/blog/2016/11/27/ovn-part1/
* http://networkop.co.uk/blog/2016/12/10/ovn-part2/
* https://blog.russellbryant.net/2016/12/19/comparing-openstack-neutron-ml2ovs-and-ovn-control-plane/
* https://blog.russellbryant.net/2016/11/11/ovn-logical-flows-and-ovn-trace/
* https://blog.russellbryant.net/2016/09/29/ovs-2-6-and-the-first-release-of-ovn/
* http://galsagie.github.io/2015/11/23/ovn-l3-deepdive/
* http://blog.russellbryant.net/2015/10/22/openstack-security-groups-using-ovn-acls/
* http://galsagie.github.io/sdn/openstack/ovs/2015/05/30/ovn-deep-dive/
* http://blog.russellbryant.net/2015/05/14/an-ez-bake-ovn-for-openstack/
* http://galsagie.github.io/sdn/openstack/ovs/2015/04/26/ovn-containers/
* http://blog.russellbryant.net/2015/04/21/ovn-and-openstack-status-2015-04-21/
* http://blog.russellbryant.net/2015/04/08/ovn-and-openstack-integration-development-update/



Install & Configuration
=======================

The ``networking-ovn`` repository includes integration with DevStack that
enables creation of a simple Open Virtual Network (OVN) development and test
environment. This document discusses what is required for manual installation
or integration into a production OpenStack deployment tool of conventional
architectures that include the following types of nodes:

* Controller - Runs OpenStack control plane services such as REST APIs
  and databases.

* Network - Runs the layer-2, layer-3 (routing), DHCP, and metadata agents
  for the Networking service. Some agents optional. Usually provides
  connectivity between provider (public) and project (private) networks
  via NAT and floating IP addresses.

  .. note::

     Some tools deploy these services on controller nodes.

* Compute - Runs the hypervisor and layer-2 agent for the Networking
  service.

Packaging
---------

Open vSwitch (OVS) includes OVN beginning with version 2.5 and considers
it experimental. The Networking service integration for OVN uses an
independent package, typically ``networking-ovn``.

Building OVS from source automatically installs OVN. For deployment tools
using distribution packages, the ``openvswitch-ovn`` package for RHEL/CentOS
and compatible distributions automatically installs ``openvswitch`` as a
dependency. Ubuntu/Debian includes ``ovn-central``, ``ovn-host``,
``ovn-docker``, and ``ovn-common`` packages that pull in the appropriate Open
vSwitch dependencies as needed.

A ``python-networking-ovn`` RPM may be obtained for Fedora or CentOS from
the RDO project.  A package based on the ``master`` branch of
``networking-ovn`` can be found at https://trunk.rdoproject.org/.

Fedora and CentOS RPM builds of OVS and OVN from the ``master`` branch of
``ovs`` can be found in this COPR repository:
https://copr.fedorainfracloud.org/coprs/leifmadsen/ovs-master/.

Controller nodes
----------------

Each controller node runs the OVS service (including dependent services such
as ``ovsdb-server``) and the ``ovn-northd`` service. However, only a single
instance of the ``ovsdb-server`` and ``ovn-northd`` services can operate in
a deployment. However, deployment tools can implement active/passive
high-availability using a management tool that monitors service health
and automatically starts these services on another node after failure of the
primary node. See the :ref:`faq` for more information.

#. Install the ``openvswitch-ovn`` and ``networking-ovn`` packages.

#. Start the OVS service. The central OVS service starts the ``ovsdb-server``
   service that manages OVN databases.

   Using the *systemd* unit:

   .. code-block:: console

      # systemctl start openvswitch

   Using the ``ovs-ctl`` script:

   .. code-block:: console

      # /usr/share/openvswitch/scripts/ovs-ctl start  --system-id="random"

#. Configure the ``ovsdb-server`` component. By default, the ``ovsdb-server``
   service only permits local access to databases via Unix socket. However,
   OVN services on compute nodes require access to these databases.

   * Permit remote database access.

     .. code-block:: console

        # ovs-appctl -t ovsdb-server ovsdb-server/add-remote ptcp:6640:IP_ADDRESS

     Replace ``IP_ADDRESS`` with the IP address of the management network
     interface on the controller node.

     .. note::

        Permit remote access to TCP port 6640 on any host firewall.

#. Start the ``ovn-northd`` service.

   Using the *systemd* unit:

   .. code-block:: console

      # systemctl start ovn-northd

   Using the ``ovn-ctl`` script:

   .. code-block:: console

      # /usr/share/openvswitch/scripts/ovn-ctl start_northd

   Options for *start_northd*:

   .. code-block:: console

      # /usr/share/openvswitch/scripts/ovn-ctl start_northd --help
      # ...
      # DB_NB_SOCK="/usr/local/etc/openvswitch/nb_db.sock"
      # DB_NB_PID="/usr/local/etc/openvswitch/ovnnb_db.pid"
      # DB_SB_SOCK="usr/local/etc/openvswitch/sb_db.sock"
      # DB_SB_PID="/usr/local/etc/openvswitch/ovnsb_db.pid"
      # ...

#. Configure the Networking server component. The Networking service
   implements OVN as an ML2 driver. Edit the ``/etc/neutron/neutron.conf``
   file:

   * Enable the ML2 core plug-in.

     .. code-block:: ini

        [DEFAULT]
        ...
        core_plugin = neutron.plugins.ml2.plugin.Ml2Plugin

   * Enable the OVN layer-3 service.

     .. code-block:: ini

        [DEFAULT]
        ...
        service_plugins = networking_ovn.l3.l3_ovn.OVNL3RouterPlugin

#. Configure the ML2 plug-in. Edit the
   ``/etc/neutron/plugins/ml2/ml2_conf.ini`` file:

   * Configure the OVN mechanism driver, network type drivers, self-service
     (tenant) network types, and enable the port security extension.

     .. code-block:: ini

        [ml2]
        ...
        mechanism_drivers = ovn
        type_drivers = local,flat,vlan,geneve
        tenant_network_types = geneve
        extension_drivers = port_security
        overlay_ip_version = 4

     .. note::

        To enable VLAN self-service networks, add ``vlan`` to the
        ``tenant_network_types`` option. The first network type
        in the list becomes the default self-service network type.

        To use IPv6 for all overlay (tunnel) network endpoints,
        set the ``overlay_ip_version`` option to ``6``.

   * Configure the Geneve ID range and maximum header size. The IP version
     overhead (20 bytes for IPv4 (default) or 40 bytes for IPv6) is added
     to the maximum header size based on the ML2 ``overlay_ip_version``
     option.

     .. code-block:: ini

        [ml2_type_geneve]
        ...
        vni_ranges = 1:65536
        max_header_size = 38

     .. note::

        The Networking service uses the ``vni_ranges`` option to allocate
        network segments. However, OVN ignores the actual values. Thus, the ID
        range only determines the quantity of Geneve networks in the
        environment. For example, a range of ``5001:6000`` defines a maximum
        of 1000 Geneve networks.

   * Optionally, enable support for VLAN provider and self-service
     networks on one or more physical networks. If you specify only
     the physical network, only administrative (privileged) users can
     manage VLAN networks. Additionally specifying a VLAN ID range for
     a physical network enables regular (non-privileged) users to
     manage VLAN networks. The Networking service allocates the VLAN ID
     for each self-service network using the VLAN ID range for the
     physical network.

     .. code-block:: ini

        [ml2_type_vlan]
        ...
        network_vlan_ranges = PHYSICAL_NETWORK:MIN_VLAN_ID:MAX_VLAN_ID

     Replace ``PHYSICAL_NETWORK`` with the physical network name and
     optionally define the minimum and maximum VLAN IDs. Use a comma
     to separate each physical network.

     For example, to enable support for administrative VLAN networks
     on the ``physnet1`` network and self-service VLAN networks on
     the ``physnet2`` network using VLAN IDs 1001 to 2000:

     .. code-block:: ini

        network_vlan_ranges = physnet1,physnet2:1001:2000

   * Enable security groups.

     .. code-block:: ini

        [securitygroup]
        ...
        enable_security_group = true

     .. note::

        The ``firewall_driver`` option under ``[securitygroup]`` is ignored
        since the OVN ML2 driver itself handles security groups.

   * Configure OVS database access and L3 scheduler

     .. code-block:: ini

        [ovn]
        ...
        ovn_nb_connection = tcp:IP_ADDRESS:6641
        ovn_sb_connection = tcp:IP_ADDRESS:6642
        ovn_l3_scheduler = OVN_L3_SCHEDULER

     .. note::

        Replace ``IP_ADDRESS`` with the IP address of the controller node that
        runs the ``ovsdb-server`` service. Replace ``OVN_L3_SCHEDULER`` with
        ``leastloaded`` if you want the scheduler to select a compute node with
        the least number of gateway ports or ``chance`` if you want the
        scheduler to randomly select a compute node from the available list of
        compute nodes.

#. Start the ``neutron-server`` service.

Network nodes
-------------

Deployments using OVN native layer-3 and DHCP services do not require
conventional network nodes because connectivity to external networks
(including VTEP gateways) and routing occurs on compute nodes.

Compute nodes
-------------

Each compute node runs the OVS and ``ovn-controller`` services. The
``ovn-controller`` service replaces the conventional OVS layer-2 agent.

#. Install the ``openvswitch-ovn`` and ``networking-ovn`` packages.

#. Start the OVS service.

   Using the *systemd* unit:

   .. code-block:: console

      # systemctl start openvswitch

   Using the ``ovs-ctl`` script:

   .. code-block:: console

      # /usr/share/openvswitch/scripts/ovs-ctl start --system-id="random"

#. Configure the OVS service.

   * Use OVS databases on the controller node.

     .. code-block:: console

        # ovs-vsctl set open . external-ids:ovn-remote=tcp:IP_ADDRESS:6642

     Replace ``IP_ADDRESS`` with the IP address of the controller node
     that runs the ``ovsdb-server`` service.

   * Enable one or more overlay network protocols. At a minimum, OVN requires
     enabling the ``geneve`` protocol. Deployments using VTEP gateways should
     also enable the ``vxlan`` protocol.

     .. code-block:: console

        # ovs-vsctl set open . external-ids:ovn-encap-type=geneve,vxlan

     .. note::

        Deployments without VTEP gateways can safely enable both protocols.

   * Configure the overlay network local endpoint IP address.

     .. code-block:: console

        # ovs-vsctl set open . external-ids:ovn-encap-ip=IP_ADDRESS

     Replace ``IP_ADDRESS`` with the IP address of the overlay network
     interface on the compute node.

#. Start the ``ovn-controller`` service.

   Using the *systemd* unit:

   .. code-block:: console

      # systemctl start ovn-controller

   Using the ``ovn-ctl`` script:

   .. code-block:: console

      # /usr/share/openvswitch/scripts/ovn-ctl start_controller

Verify operation
----------------

#. Each compute node should contain an ``ovn-controller`` instance.

   .. code-block:: console

      # ovn-sbctl show
        <output>
