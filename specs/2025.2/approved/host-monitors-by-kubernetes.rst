..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.
 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
host monitors by kubernetes
==========================================

https://blueprints.launchpad.net/masakari/+spec/host-monitors-by-kubernetes


Problem description
===================

When using Openstack on Kubernetes, the current host monitoring process
lacks efficiency and simplicity, requiring additional software such as
Consul or Pacemaker.

Proposed change
===============

This spec is mainly to add a new host monitoring driver to masakari-monitors
using the Kubernetes-client. By leveraging the Kubernetes API, kubernetes-native
openstack operator can efficiently retrieve and monitor the status of the host
(node) without the complexities associated with external configuration tools like
Consul or Pacemaker.

Administrator can set the host monitoring driver to Kubernetes in the host
configuration.


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

This periodically calls Kubernetes-api based on the value of monitoring_interval.


Other deployer impact
---------------------

"kubernetes" is added to masakari-monitors configuration as the new host.monitoring_driver
type for kubernetes-native openstack operator


Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
* do-gyun kim <dogyun7949@gmail.com>


Work Items
----------

* Create a new monitoring driver using the kubernetes-client.

* Update docs about host monitors.

* Add unit tests.


Dependencies
============

This feature require python kubernetes-client library.


Testing
=======

Add required unit tests which will run in gate.


Documentation Impact
====================

Update masakari-hostmonitor reference documentation.


References
==========

None


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - 2025.2 Flamingo
     - Introduced