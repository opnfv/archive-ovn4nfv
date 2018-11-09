.. _ovn4nfv-releasenotes:

.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) Open Platform for NFV Project, Inc. and its contributors

========
Abstract
========

This document compiles the release notes for OPNFV release when using Apex,
Joid and Fuel as a deployment tool.

===============
Important Notes
===============

This notes provides release information for the use of Apex, Joid and Fuel
as deployment tool for the Gambia release of OPNFV.

The goal of the Gambia release and this Apex and Fuel based deployment
process is to establish a lab ready platform accelerating further development
of the OPNFV infrastructure.

Carefully follow the installation-instructions.

=======
Summary
=======

For Gambia release, OVN4NFV is supported with APEX and FUEL installers.

This Gambia artifact provides Apex and Fuel as the deployment stage tool in the
OPNFV CI pipeline including:

- Documentation built by Jenkins

  - overall OPNFV documentation

  - this document (release notes)

  - installation instructions

- Automated validation of the Fraser deployment.

============
Release Data
============

+--------------------------------------+--------------------------------------+
| **Project**                          | ovn4nfv                              |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Repo/tag**                         | ovn4nfv/opnfv-7.0                    |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Release designation**              | Gambia 7.0                           |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Release date**                     | November 09 2018                     |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Purpose of the delivery**          | New Scenario PoC: OVN SFC            |
|                                      | Scenario documentation, OVN-CN-VM    |
|                                      | OVN support in Fuel installer.       |
+--------------------------------------+--------------------------------------+


Deliverables
============

Software Deliverables
---------------------

- `Apex based installation <https://git.opnfv.org/apex>`_

- `Fuel based installation <https://git.opnfv.org/fuel>`_

Documentation Deliverables
--------------------------

- `Installation instructions <https://git.opnfv.org/ovn4nfv/tree/docs/development/openstack-networking-ovn.rst?h=stable/gambia>`_

- Release notes (This document)

- `User guide and Testing notes <https://git.opnfv.org/ovn4nfv/tree/docs/testing/testing-notes.rst?h=stable/gambia>`_

- `Scenario Documentation (K8S-OVN) <https://git.opnfv.org/ovn4nfv/tree/docs/scenarios/JOID/k8s-ovn-lb-noha.rst?h=stable/gambia>`_

- `Scenario Documentation (OVN-SFC) <https://git.opnfv.org/ovn4nfv/tree/docs/development/ovn-sfc-openstack.rst?h=stable/gambia>`_

==========
References
==========
For more information, please see:

OPNFV
=====

1) `OPNFV Home Page <http://www.opnfv.org>`_
2) `OPNFV Documentation <http://docs.opnfv.org>`_
3) `OPNFV Software Downloads <https://www.opnfv.org/software/download>`_
4) `OVN4NFV Project <https://wiki.opnfv.org/display/PROJ/Ovn4nfv>`_

OpenStack
=========

1) `networking-ovn Project <https://docs.openstack.org/networking-ovn/latest>`_
2) `OVN with K8S <https://github.com/openvswitch/ovn-kubernetes>`_
3) `OVN Architecture <http://openvswitch.org/support/dist-docs/ovn-architecture.7.html>`_
