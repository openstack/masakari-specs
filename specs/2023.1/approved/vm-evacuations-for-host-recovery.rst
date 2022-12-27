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

If one compute node failed, Masakari will evacuate the instances from
the failed host.

If a large number of hosts fail at the same time, the resources of
computing nodes are dramatically reduced. There would not be enough
resources for all instances to recovery. So it is reasonable that
the very important instances to be firstly evacuated, and evacuations
can be aborted once the cloud environment encounters an irreversible
condition.

When the failed hosts come back, the restored resources may be lying
idle. In order to make full use of the restored resources, It needs
to move instances to the restored hosts. Sometimes there may be a
distribution on purpose. The vm moves automatically such as DRS
or manually could mess up the distribution. So it is a good idea to
save the evaucations when the host is failed, and move instances back
when the host is restored according to the previous evaucations.

Proposed change
===============

This spec is mainly to record vm moves information in
the database, mainly including instance_uuid, notification_uuid,
source_host, dest_host, type, status, start_time and end_time.

User can get vm moves information of a 'COMPUTE_HOST' type
notification by vmove API.

Alternatives
------------

None

Data model impact
-----------------

The table ``vmoves`` will be added into the Masakari database.

* created_at: Datetime.
* updated_at: Datetime.
* deleted_at: Datetime.
* deleted: Boolean.
* uuid: UUID. UUID of the vmove.
* notification_uuid: UUID. UUID of notification the vmove belong to.
* instance_uuid: UUID. UUID of instance.
* instance_uuid: String. Name of instance.
* source_host: String. Source host name of the vmove.
* dest_host: String. Destination host name of the vmove.
* start_time: Datetime. Start time of the vmove.
* end_time: Datetime. End time of the vmove.
* type: String. Represents possible types for the vmove, such as
  migration, live_migration or evacuation.
* status: String. Represents possible statuses for the vmove, such as
  pending, ongoing, ignored, failed or succeeded.
* message: String. Display some meaningful information if the vmove is
  failed or ignored.

REST API impact
---------------

Following vmove API will be introduced in a new API micro-version.

* GET /notifications/<notification_id>/vmoves

  response example::

    {
        "vmoves": [
            {
                "uuid": "239f95ca-fd46-44d2-8ff8-35e8a9c94f69",
                "instance_uuid": "33826ebd-af0f-445d-833f-e06340f7ae1c",
                "instance_name": "vm-1",
                "notification_uuid": "c0fa1a39-c150-4b86-ae97-8fae31700c67",
                "source_host": "node01",
                "dest_host": "node02",
                "start_time": "2022-11-22 14:50:22",
                "end_time": "2022-11-22 14:50:35",
                "type": "evacuation",
                "status": "succeeded",
                "message": null
            },
            {
                "uuid": "65a5da84-5819-4aea-8278-a28d2b489028",
                "instance_uuid": "e1a5a45b-f251-47cf-9c5f-fa1e66e1286a",
                "instance_name": "vm-2",
                "notification_uuid": "c0fa1a39-c150-4b86-ae97-8fae31700c67",
                "source_host": "node01",
                "dest_host": "node02",
                "start_time": "2022-11-22 14:50:23",
                "end_time": "2022-11-22 14:50:38",
                "type": "evacuation",
                "status": "succeeded",
                "message": null
            }
        ]
    }

* GET /notifications/<notification_id>/vmoves/<vmove_id>

  response example::

   {
        "vmove":
            {
                "uuid": "239f95ca-fd46-44d2-8ff8-35e8a9c94f69",
                "instance_uuid": "33826ebd-af0f-445d-833f-e06340f7ae1c",
                "instance_name": "vm-1",
                "notification_uuid": "c0fa1a39-c150-4b86-ae97-8fae31700c67",
                "source_host": "node01",
                "dest_host": "node02",
                "start_time": "2022-11-22 14:50:22",
                "end_time": "2022-11-22 14:50:38",
                "type": "evacuation",
                "status": "succeeded",
                "message": null
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

The masakari-dashboard and openstacksdk will be updated to support
vm moves for host type notification in a new micro-version.

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

* suzhengwei <suzhengwei@inspur.com>

Work Items
----------

* Create the object definition, database schema, updating
  engine to handle this.

* Create a new API microversion to get information for all vmoves
  and get detailed information about a particular vmove.

* Update docs about vm moves for host recovery

* Update masakari-dashboard and openstacksdk to manage vm moves.

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
   * - Yoga
     - Re-proposed
