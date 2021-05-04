..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
vm evacuations for host recovery
=================================

https://blueprints.launchpad.net/masakari/+spec/vm-evacuations-for-host-recovery


Problem description
===================

If one compute node failed, Masakari will evacuate the instances
from the failed host.

Generally, the resources of computing nodes are gradually reduced.
If a large number of hosts fail at the same time and there are not
enough computing node resources, the operator needs to set the
priority of instance evacuation in advance to ensure that the
evacuation can be carried out in the order of priority.

When the failed compute node recovers, in order to make full
use of the computing node resources and due to some instances
needing to run on a specific computing node, the operator wants
to migrate the instance back to the original node.

Proposed change
===============

This spec is mainly to record instance evacuation information in
the database, provide two interfaces to support obtaining all
evacuation information lists and specific evacuation information
details, and prepare relevant information for supporting the
migration of the instances back to previously failed hosts.

* Record instance evacuation information in the database, mainly
  including instance_id, notification_id, source_host, dest_host,
  status. The status is pending.
* User can get evacuation information about a specific masakari
  notification by ``GET /notifications/<notification_id>/evacuations``
  API.
* User can get detailed information about a specific evacuation record
  of a particular masakari notification by
  ``GET /notifications/<notification_id>/evacuations/<evacuation_id>``
  API.


Alternatives
------------

None

Data model impact
-----------------

The table ``evacuation`` will be added into the Masakari database.

* created_at: Datetime.
* updated_at: Datetime.
* deleted_at: Datetime.
* deleted: Boolean.
* uuid: UUID. uuid of evacuation record.
* notification_uuid: UUID. uuid of notification.
* instance_uuid: UUID. uuid of instance.
* source_host_name: String. The source compute node before the instance
  evacuated.
* dest_host_name: String. The destination compute node after the instance is
  evacuated.
* status: String. Represents possible statuses for notifications, such as
  pending, ongoing, ignored, failed and succeeded.
* status_details: String. Store the details reason of evacuate failed/ignored.
* priority: Numeric. Set the evacuation priority and support the
  evacuation of instances in order. The default value is 1.

REST API impact
---------------

Following changes will be introduced in a new API micro-version.

* GET /notifications/<notification_id>/evacuations

  response example::

    {
        "evacuations": [
            {
                "uuid": "239f95ca-fd46-44d2-8ff8-35e8a9c94f69",
                "instance_uuid": "33826ebd-af0f-445d-833f-e06340f7ae1c",
                "notification_uuid": "c0fa1a39-c150-4b86-ae97-8fae31700c67",
                "source_host_name": "node01",
                "dest_host_name": "node02",
                "status": "pending",
                "status_details": "",
                "priority": "1"
            }
        ]
    }

* GET /notifications/<notification_id>/evacuations/<evacuation_id>

  response example::

   {
        "evacuation":
            {
                "uuid": "239f95ca-fd46-44d2-8ff8-35e8a9c94f69",
                "instance_uuid": "33826ebd-af0f-445d-833f-e06340f7ae1c",
                "notification_uuid": "c0fa1a39-c150-4b86-ae97-8fae31700c67",
                "source_host_name": "node01",
                "dest_host_name": "node02",
                "status": "pending",
                "status_details": "",
                "priority": "1"
            }
    }

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

The python-masakariclient, masakari-dashboard and openstacksdk will be updated
to support instance evacuations for host recovery in a new micro-version.

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

* shenxinxin <shenxinxin@inspur.com>
* suzhengwei <sugar-2008@163.com>

Work Items
----------

* Create the object definition, database schema, updating
  engine to handle this.

* Create a new API microversion to get information for all evacuations
  and get detailed information about a particular evacuation.

* Update docs for instance evacuations for host recovery

* Update python-masakariclient, masakari-dashboard and openstacksdk to
  manage instance evacuations for host recovery.

* Add unit and functional tests.

Dependencies
============

None

Testing
=======

Unit and functional test is neccessary.

Add required unit and functional tests which will run in gate.

Documentation Impact
====================

Update Masakari API reference documentation.

References
==========

None

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Xena
     - Introduced
