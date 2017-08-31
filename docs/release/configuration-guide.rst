Devstack Configuration
======================

Devstack - localrc/local.conf
-----------------------------

Sample DevStack local.conf.

.. code-block:: console

  #
  # This sample file is intended to be used for your typical DevStack environment
  # that's running all of OpenStack on a single host.  This can also be used as
  # the first host of a multi-host test environment.
  #
  # No changes to this sample configuration are required for this to work.
  #
  DATABASE_PASSWORD=password
  RABBIT_PASSWORD=password
  SERVICE_PASSWORD=password
  SERVICE_TOKEN=password
  ADMIN_PASSWORD=password

  # The DevStack plugin defaults to using the ovn branch from the official ovs
  # repo.  You can optionally use a different one.  For example, you may want to
  # use the latest patches in blp's ovn branch:
  #OVN_REPO=https://github.com/blp/ovs-reviews.git
  #OVN_BRANCH=ovn

  enable_plugin networking-ovn https://git.openstack.org/openstack/networking-ovn
  enable_service ovn-northd
  enable_service ovn-controller
  enable_service networking-ovn-metadata-agent

  # Use Neutron instead of nova-network
  disable_service n-net
  enable_service q-svc

  # Disable Neutron agents not used with OVN.
  disable_service q-agt
  disable_service q-l3
  disable_service q-dhcp
  disable_service q-meta

  # Horizon (the web UI) is enabled by default. You may want to disable
  # it here to speed up DevStack a bit.
  enable_service horizon
  #disable_service horizon

  # Cinder (OpenStack Block Storage) is disabled by default to speed up
  # DevStack a bit. You may enable it here if you would like to use it.
  disable_service cinder c-sch c-api c-vol
  #enable_service cinder c-sch c-api c-vol

  # How to connect to ovsdb-server hosting the OVN NB database.
  #OVN_NB_REMOTE=tcp:$SERVICE_HOST:6641

  # How to connect to ovsdb-server hosting the OVN SB database.
  #OVN_SB_REMOTE=tcp:$SERVICE_HOST:6642

  # A UUID to uniquely identify this system.  If one is not specified, a random
  # one will be generated and saved in the file 'ovn-uuid' for re-use in future
  # DevStack runs.
  #OVN_UUID=

  # If using the OVN native layer-3 service, choose a router scheduler to
  # manage the distribution of router gateways on hypervisors/chassis.
  # Default value is leastloaded.
  #OVN_L3_SCHEDULER=leastloaded

  # Whether or not to build custom openvswitch kernel modules from the ovs git
  # tree. This is enabled by default.  This is required unless your distro kernel
  # includes ovs+conntrack support.  This support was first released in Linux 4.3,
  # and will likely be backported by some distros.
  #OVN_BUILD_MODULES=False

  # Enable services, these services depend on neutron plugin.
  #enable_plugin neutron https://git.openstack.org/openstack/neutron
  #enable_service q-qos
  #enable_service q-trunk

  # Skydive
  #enable_plugin skydive https://github.com/redhat-cip/skydive.git
  #enable_service skydive-analyzer
  #enable_service skydive-agent

  # If you want to enable a provider network instead of the default private
  # network after your DevStack environment installation, you *must* set
  # the Q_USE_PROVIDER_NETWORKING to True, and also give FIXED_RANGE,
  # NETWORK_GATEWAY and ALLOCATION_POOL option to the correct value that can
  # be used in your environment. Specifying Q_AGENT is needed to allow devstack
  # to run various "ip link set" and "ovs-vsctl" commands for the provider
  # network setup.
  #Q_AGENT=openvswitch
  #Q_USE_PROVIDER_NETWORKING=True
  #PHYSICAL_NETWORK=providernet
  #PROVIDER_NETWORK_TYPE=flat
  #PUBLIC_INTERFACE=<public interface>
  #OVS_PHYSICAL_BRIDGE=br-provider
  #PROVIDER_SUBNET_NAME=provider-subnet
  # use the following for IPv4
  #IP_VERSION=4
  #FIXED_RANGE=<CIDR for the Provider Network>
  #NETWORK_GATEWAY=<Provider Network Gateway>
  #ALLOCATION_POOL=<Provider Network Allocation Pool>
  # use the following for IPv4+IPv6
  #IP_VERSION=4+6
  #FIXED_RANGE=<CIDR for the Provider Network>
  #NETWORK_GATEWAY=<Provider Network Gateway>
  #ALLOCATION_POOL=<Provider Network Allocation Pool>
  # IPV6_PROVIDER_FIXED_RANGE=<v6 CDIR for the Provider Network>
  # IPV6_PROVIDER_NETWORK_GATEWAY=<v6 Gateway for the Provider Network>

  # If you wish to use the provider network for public access to the cloud,
  # set the following
  #Q_USE_PROVIDERNET_FOR_PUBLIC=True
  #PUBLIC_NETWORK_NAME=<Provider network name>
  #PUBLIC_NETWORK_GATEWAY=<Provider network gateway>
  #PUBLIC_PHYSICAL_NETWORK=<Provider network name>
  #IP_VERSION=4
  #PUBLIC_SUBNET_NAME=<provider subnet name>
  #Q_FLOATING_ALLOCATION_POOL=<Provider Network Allocation Pool>
  #FLOATING_RANGE=<CIDR for the Provider Network>

  # NOTE: DO NOT MOVE THESE SECTIONS FROM THE END OF THIS FILE
  # IF YOU DO, THEY WON'T WORK!!!!!
  #

  # Enable Nova automatic host discovery for cell every 2 seconds
  # Only needed in case of multinode devstack, as otherwise there will be issues
  # when the 2nd compute node goes online.
  discover_hosts_in_cells_interval=2


Neutron - metadata_agent.ini
--------------------------

The following configuration options in /etc/neutron/metadata_agent.ini
are required when OVN is enabled in OpenStack neutron.

.. code-block:: console

  ...
  ...
  [ovs]
  #
  # From networking_ovn.metadata.agent
  #

  # The connection string for the native OVSDB backend.
  # Use tcp:IP:PORT for TCP connection.
  # Use unix:FILE for unix domain socket connection. (string value)
  #ovsdb_connection = unix:/usr/local/var/run/openvswitch/db.sock

  # Timeout in seconds for the OVSDB connection transaction (integer value)
  #ovsdb_connection_timeout = 180

  [ovn]
  ovn_sb_connection = tcp:<controller-ip>:<port>


Neutron - neutron.conf
--------------------

The following configuration changes are required in /etc/neutron/neutron.conf

.. code-block:: console

  [DEFAULT]
  service_plugins = networking_ovn.l3.l3_ovn.OVNL3RouterPlugin

Neutron -  ml2_conf.ini
--------------------

The following configuration changes are required in /etc/neutron/plugins/ml2/ml2_conf.ini

.. code-block:: console

  [ml2]
  mechanism_drivers = ovn,logger

  [ovn]
  ovn_metadata_enabled = True
  ovn_l3_scheduler = leastloaded
  neutron_sync_mode = log
  ovn_sb_connection = tcp:<controller-ip>:<port>
  ovn_nb_connection = tcp:<controller-ip>:<port>
