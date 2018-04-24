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
as deployment tool for the Fraser release of OPNFV.

The goal of the Fraser release and this Apex, Fuel and Joid based deployment
process is to establish a lab ready platform accelerating further development
of the OPNFV infrastructure.

Carefully follow the installation-instructions.

=======
Summary
=======

For Fraser release, OVN4NFV is supported with APEX, JOID and FUEL installers.

This Fraser artifact provides Apex, Joind and Fuel as the deployment stage tool in the
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
| **Repo/tag**                         | opnfv-6.0.0                          |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Release designation**              | Fraser 6.0                           |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Release date**                     | April 27 2018                        |
|                                      |                                      |
+--------------------------------------+--------------------------------------+
| **Purpose of the delivery**          | Bug fixes, Document improvements,    |
|                                      | Scenario documentation,              |
|                                      | OVN support in Fuel installer.       |
+--------------------------------------+--------------------------------------+


Deliverables
============

Software Deliverables
---------------------

- `Apex based installation <https://git.opnfv.org/apex>`_

- `Joid based installation <https://git.opnfv.org/joid>`_

- `Fuel based installation <https://git.opnfv.org/fuel>`_

Documentation Deliverables
--------------------------

- `Installation instructions <https://git.opnfv.org/ovn4nfv/tree/docs/development/openstack-networking-ovn.rst?h=stable/fraser>`_

- Release notes (This document)

- `User guide and Testing notes <https://git.opnfv.org/ovn4nfv/tree/docs/testing/testing-notes.rst?h=stable/fraser>`_

- `Scenario Documentation (JOID - K8S-OVN) <https://git.opnfv.org/ovn4nfv/tree/docs/scenarios/JOID/k8s-ovn-lb-noha.rst?h=stable/fraser>`_


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
