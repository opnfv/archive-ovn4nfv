OVN4NFV Project
===============
OVN complements the existing capabilities of OVS to add native support for
virtual network abstractions such as virtual L2 & L3 overlays, L3 routing
and security groups. Instead of treating ovsdb and Open Flow actions
separately, OVN provides simpler interface for managing virtual networks.

Besides the simpler interface, OVN takes care of transforming simple flow
rules of virtual network to complex Open Flow rules on the Open vSwitches
involved. The Openstack project networking-ovn implements the neutron api
using OVN. As part of ovn4nfv project we would like to enable OVN along
with the openstack neutron plugin networking-ovn as a deployable network
control component in the OPNFV build. This would make it easier to manage
virtual networks and push more of network intelligence to the edge onto the
compute nodes. Since OVN has inherent support for containers, this would
allow OPNFV to orchestrate container VNFs.

Further this will make the controller architecture much more simpler and
scalable by placing the controller (ovn-controller) next to the Open vSwitch.

OVN for OPNFV at OPNFV Design Summit, Nov, 2015
  - https://wiki.opnfv.org/_media/events/ovn-opnfv-summit2015.pdf

