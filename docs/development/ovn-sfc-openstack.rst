===================
OVN-SFC POC Details
===================

Purpose
=======
The purpose of this Proof-of-concept is to showcase Service Function
Chaining with OVN.

Scope
=====

The Scope of this document is to describe SFC using OVN and discuss
installation and configuration of OVN to instantiate a forwarding path.

Steps
=====
1. Install CentOS7 minimal install:
-----------------------------------
- Make sure to enable network interface.
- Just create a root password. Don't create any users

2. Create user:
---------------
Below are the instructions to create user - stack, for use with Devstack.

- $ sudo useradd -s /bin/bash -d /opt/stack -m stack
- $ echo 'stack ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/stack
- $ sudo su - stack

3. Install git
--------------
- $ sudo yum install git -y

4. clone Devstack and Networking-ovn
------------------------------------
- $ git clone http://git.openstack.org/openstack-dev/devstack.git
- $ git clone http://git.openstack.org/openstack/networking-ovn.git
- $ cd devstack
- $ cp ../networking-ovn/devstack/local.conf.sample local.conf

5. Edit the local.conf file:
----------------------------
- Add (uncomment and edited)

  - OVN_REPO=https://github.com/doonhammer/ovs
  - OVN_BRANCH=sfc.v30
- Uncomment the below line

  - OVN_BUILD_MODULES=False

We use forked/modifed OVS for SFC usecase from John McDowall.

6. Devstack Preliminaries:
--------------------------
- $ ./stack.sh
- $ . ~/devstack/openrc admin
- $ openstack keypair create demo &amp;gt; ~/id_rsa_demo
- $ chmod 600 ~/id_rsa_demo
- $ for group in $(openstack security group list -f value -c ID);
       do openstack security group rule create --ingress --ethertype IPv4 --dst-port 22 --protocol tcp $group;
          openstack security group rule create --ingress --ethertype IPv4 -- protocol ICMP $group;
       done
- $ IMAGE_ID=$(openstack image list -f value -c ID)

10. Create Neutron network and subnet
--------------------------------------
- $ openstack network create --project admin --provider-network-type geneve n1
- $ openstack subnet create --subnet-range 10.1.1.0/24 --network n1 n1subnet


10. Spawn VMs
-------------
- Create 5 VMs, 3 VMs to act as communication end-points (a,b, and c) and two
  VMs to act as VNFs (vnf1 &amp; vnf2).
- The 2 VNF VMs are created with two NICs to act as ingress and egress ports
  (Optional)
- Created two SFCs:
  - SFC1: any traffic from VM a to VM b will go through vnf1
  - SFC1: any traffic from VM a to VM c will go through vnf2 then vnf1


A. SFC with OVN - Scenario 1:
-----------------------------

***********************
1. create VMs and VNFs:
***********************
- $ openstack server create --nic net-id=n1,v4-fixed-ip=10.1.1.5 --flavor m1.nano --image $IMAGE_ID --key-name demo a
- $ openstack server create --nic net-id=n1,v4-fixed-ip=10.1.1.6 --flavor m1.nano --image $IMAGE_ID --key-name demo b
- $ openstack server create --nic net-id=n1,v4-fixed-ip=10.1.1.10 --nic net-id=n1,v4-fixed-ip=10.1.1.11 --flavor m1.nano --image $IMAGE_ID --key-name demo vnf1
- $ openstack port set --name ap $(openstack port list --server a -f value -c ID)
- $ openstack port set --name bp $(openstack port list --server b -f value -c ID)
- $ AP_MAC=$(openstack port show -f value -c mac_address ap)
- $ BP_MAC=$(openstack port show -f value -c mac_address bp)
- $ openstack port set --name vnf1-pin $(openstack port list --server vnf1 --mac-address  fa:16:3e:a0:e9:70 -f value -c ID)
- $ openstack port set --name vnf1-pout $(openstack port list --server vnf1 --mac-address fa:16:3e:ae:0c:36 -f value -c ID)
- $ f1_pin_MAC=$(openstack port show -f value -c mac_address vnf1-pin)
- $ f1_pout_MAC=$(openstack port show -f value -c mac_address vnf1-pout)

***************************************
2. Create port-pairs, groups and chains
***************************************
The switch and ports UUIDs below will different in each environment.

- n1 = f1de57df-04e3-456b-85c0-64fd869507ad
- vnf1-pin = 6ec5aa3d-8440-44c9-acf3-a18914ca9b0d
- vnf1-pout = 3f558a9d-295e-4417-9646-d46b59be97d8
- ap = 0438495b-7de4-4bbb-b787-dff82615b541
- bp = 1f004846-3f38-450d-8f4a-e5ed0f7228e6
- cp = 9a72cc76-4d8d-494c-a959-8d672149c0ea
- vnf2-pin = 6a32edc7-23d4-42ed-9cf8-c6e0009da01d
- vnf2-pout =  8553b6d2-1433-4ab4-ab69-704d318b09af

**1. Configure the port pair vnf1-PP1**

- $ ovn-nbctl lsp-pair-add n1 vnf1-pin vnf1-pout vnf1-PP1 (didn't work with names)
- $ ovn-nbctl lsp-pair-add f1de57df-04e3-456b-85c0-64fd869507ad 6ec5aa3d-8440-44c9-acf3-a18914ca9b0d 3f558a9d-295e-4417-9646-d46b59be97d8 vnf1-PP1

**2. Configure the port chain PC1**

- $ ovn-nbctl lsp-chain-add n1 PC1
- $ ovn-nbctl lsp-chain-add f1de57df-04e3-456b-85c0-64fd869507ad PC1

**3. Configure the port pair group PG1 and add to port chain**

- $ ovn-nbctl lsp-pair-group-add PC1 PG1

**4. Add port pair to port chain**

- $ ovn-nbctl lsp-pair-group-add-port-pair PG1 vnf1-PP1

**5.  Add port chain to port classifier PCC1**

- $ lsp-chain-classifier-add SWITCH CHAIN PORT DIRECTION PATH [NAME] [MATCH]
- $ ovn-nbctl lsp-chain-classifier-add n1 PC1 bp 'entry-lport' 'bi-directional' PCC1 '';
- $ ovn-nbctl lsp-chain-classifier-add f1de57df-04e3-456b-85c0-64fd869507ad PC1 1f004846-3f38-450d-8f4a-e5ed0f7228e6 'entry-lport' 'bi-directional' PCC1 ''

*****************
3. Validating SFC
*****************

- $ ovn-trace n1 'inport == "ap" && eth.src == "$AP_MAC" && eth.dst == "$BP_MAC"'


B. SFC with OVN - Scenario 2:
-----------------------------

*************
1. Create VMs
*************
- $ openstack server create --nic net-id=n1,v4-fixed-ip=10.1.1.7 --flavor m1.nano --image $IMAGE_ID --key-name demo c
- $ openstack server create --nic net-id=n1,v4-fixed-ip=10.1.1.20 --nic net-id=n1,v4-fixed-ip=10.1.1.21 --flavor m1.nano --image $IMAGE_ID --key-name demo vnf2
- $ openstack port set --name cp $(openstack port list --server c -f value -c ID)
- $ CP_MAC=$(openstack port show -f value -c mac_address cp)
- $ openstack port set --name vnf2-pin $(openstack port list --server vnf2 --mac-address  fa:16:3e:ff:e5:76 -f value -c ID)
- $ openstack port set --name vnf2-pout $(openstack port list --server vnf2 --mac-address fa:16:3e:4c:a3:58 -f value -c ID)
- $ f2_pin_MAC=$(openstack port show -f value -c mac_address vnf2-pin)
- $ f2_pout_MAC=$(openstack port show -f value -c mac_address vnf2-pout)

****************
2. Configure SFC
****************

**1. Configure the port pair vnf2-PP1**

- $ ovn-nbctl lsp-pair-add n1 vnf2-pin vnf2-pout vnf2-PP1 (Didn't work with names)
- $ ovn-nbctl lsp-pair-add f1de57df-04e3-456b-85c0-64fd869507ad 6a32edc7-23d4-42ed-9cf8-c6e0009da01d 8553b6d2-1433-4ab4-ab69-704d318b09af vnf2-PP1
	
**2. Configure the port chain PC2**

- $ ovn-nbctl lsp-chain-add n1 PC2
- $ ovn-nbctl lsp-chain-add f1de57df-04e3-456b-85c0-64fd869507ad PC2

**3. Configure the port pair group PG2 and add to port chain**

- $ ovn-nbctl lsp-pair-group-add PC2 PG2
- $ ovn-nbctl lsp-pair-group-add PC2 PG3

**4. Add port pair to port chain**

- $ ovn-nbctl lsp-pair-group-add-port-pair PG2 vnf2-PP1
- $ ovn-nbctl lsp-pair-group-add-port-pair PG3 vnf1-PP1

**4. Add port chain to port classifier PCC2**

- $ ovn-nbctl lsp-chain-classifier-add n1 PC2 cp "entry-lport" "bi-directional" PCC2 ""
- $ ovn-nbctl lsp-chain-classifier-add f1de57df-04e3-456b-85c0-64fd869507ad PC2 9a72cc76-4d8d-494c-a959-8d672149c0ea "entry-lport" "bi-directional" PCC2 "";

********************
3. Validate Scenario
********************

- $ ovn-trace n1 'inport == "ap" && eth.src == "$AP_MAC" && eth.dst == "$CP_MAC"'

References:
-----------

1. http://docs.openvswitch.org/en/latest/tutorials/ovn-openstack/
2. https://gist.github.com/voyageur/a26943eced3324b302f1ffede45252bd
3. https://github.com/doonhammer/ovs
