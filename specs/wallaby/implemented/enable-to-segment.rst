..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================================
Add option to enable/disable a segment
=======================================

https://blueprints.launchpad.net/masakari/+spec/enable-to-segment


Problem description
===================

Sometimes, operators want to disable instance-ha temporarily. For example,
when we plan to update the hardware or pull the latest code on the compute
nodes where the masakari-monitors are running, we are required to stop
masakari-api and masakari-engine services so that it won't execute any ha
recovery workflow. If you forgot to stop these services, then it would
end up in a mess and you will need to spend lot of your time in recovering
instances which are messed up.

So, we need a simple solution which will allow masakari to disable instance-ha
for a temporarily period without needing operator to stop these services.

Proposed change
===============

This spec is mainly to add an option to enable/disable a segment

* Add ``enabled`` option to a segment. When creating a new segment, it will be
  enabled by default. Option ``enabled`` has a boolean value: 'True' means the
  segment is enabled, notifications of this segment will be processed; 'False'
  means the segment is disabled, notifications of this segment will be
  ignored.

* User can modify ``enabled`` option by ``PUT /segment/<segment_uuid>`` API.


Alternatives
------------

None

Data model impact
-----------------

Yes, add a new db column ``enabled`` of type Boolean with default value
``True`` to ``failover_segments`` table.

REST API impact
---------------

Following changes will be introduced in a new API micro-version.

* POST /segments

  request example::

    {
        "segment": {
            "service_type": "COMPUTE",
            "recovery_method": "AUTO",
            "name": "segment",
            "enabled": True
        }
    }

  response example::

    {
        "segment": {
            "uuid": "5fd9f925-0379-40db-a7f8-786a0b655b2a",
            "deleted": false,
            "created_at": "2017-04-21T08:59:53.991030",
            "description": null,
            "recovery_method": "AUTO",
            "updated_at": null,
            "service_type": "COMPUTE",
            "deleted_at": null,
            "id": 4,
            "name": "segment",
            "enabled": True
        }
    }

* PUT  /segments/{segment_id}

  request example::

    {
        "segment": {
            "name": "new_segment",
            "enabled": False
        }
    }

  response example::

    {
        "segment": {
            "uuid": "5fd9f925-0379-40db-a7f8-786a0b655b2a",
            "deleted": false,
            "created_at": "2017-04-21T08:59:54.000000",
            "description": null,
            "recovery_method": "AUTO",
            "updated_at": "2017-04-21T09:47:03.748028",
            "service_type": "COMPUTE",
            "deleted_at": null,
            "id": 4,
            "name": "new_segment",
            "enabled": False
        }
    }

* GET /segments

  response example::

    {
        "segments": [
            {
                "uuid": "9e800031-6946-4b43-bf09-8b3d1cab792b",
                "deleted": false,
                "created_at": "2017-04-20T10:17:17.000000",
                "description": "Segment1",
                "recovery_method": "auto",
                "updated_at": null,
                "service_type": "Compute",
                "deleted_at": null,
                "id": 1,
                "name": "segment2",
                "enabled": True
            }
        ]
    }

* GET /segments/<segment_uuid>

  response example::

    {
        "segment": {
            "uuid": "5fd9f925-0379-40db-a7f8-786a0b655b2a",
            "deleted": false,
            "created_at": "2017-04-21T08:59:53.991030",
            "description": null,
            "recovery_method": "AUTO",
            "updated_at": null,
            "service_type": "COMPUTE",
            "deleted_at": null,
            "id": 4,
            "name": "new_segment",
            "enabled": False
        }
    }


Security impact
---------------

None

Notifications impact
--------------------

Field ``enabled`` will be added into segment in masakari notifications.
For example the wire format of the ``create.segment.start`` notification
looks like the following::

    {
        "event_type": "segment.create.start",
        "message_id": "e44cb15b-dcba-409e-b0e1-9ee103b9a168",
        "payload": {
            "masakari_object.data": {
                "description": null,
                "fault": null,
                "name": "test",
                "recovery_method": "auto",
                "service_type": "compute",
                "enabled": True
            },
            "masakari_object.name": "SegmentApiPayload",
            "masakari_object.namespace": "masakari",
            "masakari_object.version": "1.1"
        },
        "publisher_id": "masakari-api:fake-mini",
        "timestamp": "2018-11-22 09:25:12.393979"
    }

Other end user impact
---------------------

The python-masakariclient, masakari-dashboard and openstacksdk will be updated
to support ``enabled`` parameter of the segment in a new micro-version.

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

* Create a new API microversion to handle ``enabled`` parameter in segments.

* Update docs for enabled to segment

* Update python-masakariclient, masakari-dashboard and openstacksdk to
  manage ``enabled`` parameter of the segment..

* Add functional tests

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
   * - Ussuri
     - Introduced
   * - Victoria
     - Re-proposed
