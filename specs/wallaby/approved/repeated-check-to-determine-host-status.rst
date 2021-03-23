..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
repeated check to determine host status
==========================================

https://blueprints.launchpad.net/masakari-monitors/+spec/retry-check-when-host-failure


Problem description
===================

If the platform has a bad network stability, judging from pacemaker and
corosync, the host status would swing between up and down. If the host
monitor react quickly in this condition, it would result in host recovery,
which is not expected.

Proposed change
===============

It is imprecise to determine host status by once check. repeated checks
is more reliable.

While the host-monitor keeps a sequence of one host' latest status. The host
status is 'down' only if the sequence of its latest status is consistently
'down'.

The length of the sequence is determined by the ``monitoring_samples``
configuration, which has a default int value 1. It means the host status is
'down' only if the once check status of the host is 'down'.

Meanwhile, the default value of the configuration ``monitoring_interval``
is suggested to set to 60 seconds, just the same as before.

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

None

Dependencies
============

None

Testing
=======

Unit tests are needed.

Documentation Impact
====================

Update user documentations.

References
==========

None

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Wallaby
     - Introduced
