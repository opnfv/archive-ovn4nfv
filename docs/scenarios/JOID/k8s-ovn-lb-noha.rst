.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. (c) <optionally add copywriters name>


Abstract
========

This document outlines the notes for deploying Kubernetes with OVN as the SDN and a load
balancer. JOID is used as the deployment tool.

Introduction
============
Juju OPNFV Infrastructure Deployer (JOID)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JOID as Juju OPNFV Infrastructure Deployer allows you to deploy different combinations of
OpenStack release and SDN solution in HA or non-HA mode. For OpenStack, JOID supports
Juno and Liberty. For SDN, it supports Openvswitch, OpenContrail, OpenDayLight, and ONOS.
In addition to HA or non-HA mode, it also supports deploying from the latest development
tree.

JOID heavily utilizes the technology developed in Juju and MAAS. Juju is a
state-of-the-art, open source, universal model for service oriented architecture and
service oriented deployments. Juju allows you to deploy, configure, manage, maintain,
and scale cloud services quickly and efficiently on public clouds, as well as on physical
servers, OpenStack, and containers. You can use Juju from the command line or through its
powerful GUI. MAAS (Metal-As-A-Service) brings the dynamism of cloud computing to the
world of physical provisioning and Ubuntu. Connect, commission and deploy physical servers
in record time, re-allocate nodes between services dynamically, and keep them up to date;
and in due course, retire them from use. In conjunction with the Juju service
orchestration software, MAAS will enable you to get the most out of your physical hardware
and dynamically deploy complex services with ease and confidence.

For more info on Juju and MAAS, please visit https://jujucharms.com/
and http://maas.ubuntu.com.

Production grade container Orchestrator - Kubernetes (K8S)
==========================================================

Kubernetes is an open-source system for automating deployment, scaling, and
management of containerized applications.

This is a Kubernetes cluster that includes logging, monitoring, and operational
knowledge. It is comprised of the following components and features:

- Kubernetes (automated deployment, operations, and scaling)
 TLS used for communication between nodes for security.
 A CNI plugin - in this scenario, we use OVN (Refer : https://github.com/openvswitch/ovn-kubernetes)
 A load balancer for HA kubernetes-master (Experimental)
 Optional Ingress Controller (on worker)
 Optional Dashboard addon (on master) including Heapster for cluster monitoring

- EasyRSA
 Performs the role of a certificate authority serving self signed certificates
 to the requesting units of the cluster.

- Etcd (distributed key value store)
 Minimum Three node cluster for reliability.

- Kubernetes deployment with Load Balancer
   .. code-block:: sh

      ./deploy.sh -m kubernetes -f lb -l custom -s ovn


Deployment Diagram
==================



               +-------------------------+
               |                         |
  +------------+  Kubeapi-load-balancer  +-----------------+
  |            |                         |                 |
  |            +------------+------------+                 |
  |                         |                              |
  |                         |                              |
  |                  +------+------+        +----------+   |
  |                  |             |        |          |   |
  |   +--------------+   EasyRSA   +--------+   etcd   |   |
  |   |              |             |        |          |   |
  |   |              +-------------+        +-----+----+   |
  |   |                                           |        |
  |   |                +---------+                |        |
  |   |                |         |                |        |
  |   |  +-------------+   OVN   +-------------+  |        |
  |   |  |             |         |             |  |        |
  |   |  |             +---------+             |  |        |
  |   |  |                                     |  |        |
  |   |  |                                     |  |        |
+-+---+--+----------+              +-----------+--+------+ |
|                   |              |                     | |
| Kubernetes-worker +-----------------Kubernetes-master  +-+
|                   |              |                     |
+-------------------+              +---------------------+



Using Kubernetes after Deployment
=================================

Once you have finished installing Kubernetes, you can use the
following command to test the deployment.

To deploy 5 replicas of the microbot web application inside the Kubernetes
cluster run the following command:

.. code-block:: bash

   juju run-action kubernetes-worker/0 microbot replicas=5

This action performs the following steps:

It creates a deployment titled 'microbots' comprised of 5 replicas defined
during the run of the action. It also creates a service named 'microbots'
which binds an 'endpoint', using all 5 of the 'microbots' pods.
Finally, it will create an ingress resource, which points at a
xip.io domain to simulate a proper DNS service.

Running the packaged example

You can run a Juju action to create an example microbot web application:

.. code-block:: bash

   $ juju run-action kubernetes-worker/0 microbot replicas=3
   Action queued with id: db7cc72b-5f35-4a4d-877c-284c4b776eb8

   $ juju show-action-output db7cc72b-5f35-4a4d-877c-284c4b776eb8
   results:
     address: microbot.104.198.77.197.xip.io
     status: completed
     timing:
     completed: 2016-09-26 20:42:42 +0000 UTC
     enqueued: 2016-09-26 20:42:39 +0000 UTC
     started: 2016-09-26 20:42:41 +0000 UTC
Note: Your FQDN will be different and contain the address of the cloud
instance.
At this point, you can inspect the cluster to observe the workload coming
online.

For deploying via juju, refer to : https://jujucharms.com/u/aakashkt/kubernetes-ovn/

Supported OVN scenarios
=======================
Name:      joid-k8s-ovn-lb-noha
Test Link: https://build.opnfv.org/ci/view/joid/job/joid-k8-ovn-lb-noha-baremetal-daily-master/

References
==========

Juju
----
- `Juju Charm store <https://jujucharms.com/>`
- `Juju documents <https://jujucharms.com/docs/stable/getting-started>`
- `Canonical Distibuytion of Kubernetes <https://jujucharms.com/canonical-kubernetes/>`

MAAS
----
- `Bare metal management (Metal-As-A-Service) <http://maas.io/get-started>`
- `MAAS API documents <http://maas.ubuntu.com/docs/>`

JOID
----
- `OPNFV JOID wiki <https://wiki.opnfv.org/joid>`
- `OPNFV JOID Get Started <https://wiki.opnfv.org/display/joid/JOID+Get+Started>`

Kubernetes
----------
- `Kubernetes Release artifacts <https://get.k8s.io/>`
- `Kubernetes documentation <https://kubernetes.io/>`