..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Implement RESERVED_HOST recovery action for host failure workflow
=================================================================

https://blueprints.launchpad.net/masakari/+spec/implement-recovery-methods

This spec talks about adding RESERVED_HOST recovery action for host
failure workflow. In masakari each failover segment has recovery_method
defined for it, so that if any of the host within that failover segment
goes down then recovery action will be executed to evacuate "HA_Enabled" or all
instances depending on "evacuate_all_instances" configuration option from that
host based on recovery_method.

What is RESERVED_HOST?

In each failover segment operators will keep some hosts as reserved by
disabling the compute service on those hosts and setting "reserved"
property of that host as "True". As a result, these hosts are not
selected by the nova scheduler while provisioning new instances or for
performing any other instance actions such as resize, migration etc.
These hosts can be used by masakari engine for evacuating the instances
from the failed host. Once the reserved host is used for evacuating
the instances it is no longer treated as reserved and nova scheduler can
use that host for scheduling the instances.

Problem description
===================

Masakari provides a driver interface for implementing the workflows
synchronously or asynchronously. Whoever wants to implement the
workflow can inherit the masakari driver and implement the workflows.

For implementing the RESERVED_HOST recovery action masakari engine
should provide list of reserved hosts associated with its failover segment
to the driver. Its then job of the driver to execute the workflow and use
this list for evacuating the instances from failed host. One of the task of
the workflow is to enable the compute service on reserved host so that
instances can be evacuated on that host. At the same time "reserved" property
of that host needs to be set to False. There is a possibility for multiple
host failures under one failover segment may take the same reserved host and
start the recovery workflows at the same time. To avoid this situation, lock
with current reserved host name will be acquired on each of the task and that
reserved host will be skipped if the lock acquired and evacuation will be done
on the next reserved host from the list.

Use Cases
---------

Operator may want to execute host_failure workflow using 'RESERVED_HOST'
recovery method.

Proposed change
===============

Masakari engine should execute the workflows synchronously only. Masakari
engine will load all the drivers. Whoever is going to implement the new driver,
it should be the responsibility of that driver to get the result of workflow
and send it back to the masakari engine. If someone wants to add a driver
which will execute the workflow on a different host and not on the same host
where masakari engine is running then they will need to design that driver
in such a way that workflow will execute on any host in asynchronous way and
send back the result to the masakari engine, so that masakari engine will
set the notification status to "ERROR" or "FINISHED" based on the results.

To implement "reserved_host" recovery method, we need to implement lock
mechanism over reserved host so that masakari-engine don't use same reserved
host for multiple failure notifications. There are two ways to implement the
lock mechanism:

1. Use oslo_concurrency.lockutils file based lock:
   Easy to implement, but cannot manage lock among multiple nodes.

2. Implement lock mechanism using Tooz:
   Operator would want to deploy multiple masakari-engine services on
   different nodes. For this purpose we recommend to use distributed locking
   mechanism provided by Tooz library. By default Tooz would be configured to
   use file locks, so everything will work as oslo_concurrency lock mechanism.
   If operator would want to run multiple masakari-engine services he/she
   would need to configure Tooz backend service and set it in masakari.conf.
   Currently most reliable Tooz backends are ZooKeeper and Redis.

As of now, in masakari file based lock (oslo_concurrency.lockutils) is already
used. Same mechanism will be used to acquire lock on reserved host. Tooz
support will be added later and all the existing locks will be migrated to
use Tooz locking mechanism.

Pros:
-----
1. No need to change current masakari engine implementation.
2. It's easy to implement other recovery actions such as "AUTO_PRIORITY" and
   "RH_PRIORITY" with this design.

Alternatives
------------

Execute the workflows asynchronously i.e. either workflow can be executed on
same host where masakari engine is running or on a different host altogether.
In this case masakari engine might not get the results from the workflow
execution and will not able to update notification status and set reserved
hosts to False.

To achieve this masakari engine will generate a callback URL based on the
notification_id and pass it to the driver. Sample callback URL will be like,

http://<host>:<port>/v1/notification/<notification_id>

Driver will further pass this URL and the required information such as
reserved host list, host name etc. to the workflow. Workflow will be
responsible to call the masakari using callback URL with notification status
and reserved hosts used by workflow as a body of the request.

Once this request is received by masakari, then based on the notification_id
it will map it to the notification from the database table and update the
status of the notification accordingly. Also masakari will get the list of
used reserved hosts in request body, so it will loop through it and set those
host's "reserved" property as False.

Rest API can be::

    method: PUT
    URL: URL that is passed to the workflow(contains notification_id)
    Body:
    result {
        notification_status: status of the notification, either 'error' or
        'finished', used_reserved_hosts: list of actually used reserved_hosts
        by workflow
    }

Pros:
-----

1. Better approach to implement new workflow drivers.

Cons:
-----
1. The other service which is going to request masakari using REST api should
   have required admin credentials to call the API.
2. Need to change current driver (taskflow) implementation to adopt this
   design.
3. Need to modify PUT api to incorporate this change.

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
  Dinesh Bhor <dinesh.bhor@nttdata.com>
  Abhishek Kekane <abhishek.kekane@nttdata.com>

Work Items
----------

* Implement RESERVED_HOST recovery_method for host_failure recovery in
  synchronous way for taskflow driver
* Add unit tests for the coverage

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

None

References
==========

http://eavesdrop.openstack.org/meetings/masakari/2016/masakari.2016-12-13-04.02.log.html
http://eavesdrop.openstack.org/meetings/masakari/2017/masakari.2017-02-07-04.01.log.html
