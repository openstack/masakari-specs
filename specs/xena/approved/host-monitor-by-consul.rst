..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================
host monitor by consul
=======================

https://blueprints.launchpad.net/masakari-monitors/+spec/host-monitor-by-consul


Problem description
===================

Usually, there are management network, tenant network and storage network
in one cloud platform. The compute nodes may have management, tenant and
storage interface to connect to these three networks.

Currently, Masakari hostmonitor uses pacemaker and pacemaker-remote to
monitor hosts' connection. Actually, it is monitoring the hosts heartbeat
through management interface. Once a host's management connectivity is
detected down, it will send notification to masakari to trigger host
failure recovery workflow.

This solution has some flaws especially when ``management`` connectivity
is down and the other two connectivity ``tenant`` and ``storage`` are up.
Users can still access their VMs without any interruptions, so there is no
need to send host failure notification in this case.


Proposed change
===============

This spec introduces a new host monitor. Specifically, host connectivity
monitoring via management, tenant and storage interfaces by consul agent.

The low-level architecture for host monitoring is shown as below:

.. image:: /images/host-monitor-by-consul.png

Each host runs three consul agents, which respectively bind management, tenant
and storage interfaces. They make up three independent consul cluster.

Consul agent runs in server mode on controller nodes, while in client mode on
compute nodes.

For example, consul cluster via management connectivity. All agents bind
management interface, and are responsible for running checks and keeping
services in sync.

Consul is built on top of Serf which provides a full gossip protocol that is
used for multiple purposes. Serf provides membership, failure detection, and
event broadcast. Consul uses gossip protocol to manage membership. If an agent
is found disconnected, it will broadcast messages to the cluster quickly.

Host-monitor periodically retrieves all consul members heath data from local
consul agents. It picks out every nodes' management, tenant and storage health
state separately, and combines them together. Then it will send notification
depending on the HA strategy - host states and the corresponding actions.
There will be a config file for user to decide the HA strategy and the
default HA strategy is as follows:

+-----------------+-----------------+-----------------+------------------+
|   management    |     tenant      |      storage    |     actions      |
+-----------------+-----------------+-----------------+------------------+
|       up        |       up        |       down      |     recovery     |
+-----------------+-----------------+-----------------+------------------+
|       up        |       down      |       down      |     recovery     |
+-----------------+-----------------+-----------------+------------------+
|       down      |       up        |       down      |     recovery     |
+-----------------+-----------------+-----------------+------------------+
|       down      |       down      |       down      |     recovery     |
+-----------------+-----------------+-----------------+------------------+

* 'up' represents connectivity up.
* 'down' represents connectivity down.
* 'recovery' represents host recovery.

User can define the HA strategy according to the physical environment.
For example, if there is only one consul cluster of management, the HA
strategy would be the same as the existing solution based on pacemaker.

+-----------------+------------------+
|   management    |     actions      |
+-----------------+------------------+
|       down      |     recovery     |
+-----------------+------------------+


Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:

* suzhengwei <sugar-2008@163.com>

Work Items
----------

- Masakari host-monitor driver based on consul

- masakari documentation updates

Dependencies
============

- Requires that consul agents are installed and running to monitor
  the hosts management, tenant and storage connectivity.

Testing
=======

Unit tests and functional tests will be needed.

Documentation Impact
====================

The admin configuration documentation need to be updated.

References
==========

https://www.consul.io/


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Victoria
     - Introduced
   * - Xena
     - Re-proposed
