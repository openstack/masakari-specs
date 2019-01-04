..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Add progress details for recovery workflows
============================================

https://blueprints.launchpad.net/masakari/+spec/progress-details-recovery-workflows

This blueprint proposes to have a feature that notifies events for recovery
workflows.

Problem description
===================

Currently, Masakari doesn't send any events during recovery operation request
received by Masakari monitor.

It would be useful to receive events at each stage of task of recovery
workflow along with completion status and progress details so that operator
will come to know about what's happening during execution.

Use Cases
---------

Operators will be able to know following things by detailed progress details
captured during each event of recovery:

* Beginning/End of each task of recovery flow
* Errors of failure of process recovery
* Progress details which will contain the details of each task


Proposed change
===============

Masakari Recovery Workflow is a certain set of tasks executed to recover
from failure. Masakari supports three types of recovery failures:

* instance-failure
* process-failure
* host-failure

For each of these failures, Masakari executes a workflow to recover from
failure. Currently Masakari uses taskflow library to execute the workflow
which consists of recovery actions which are predefined and are executed
linearly. Proposing here to record these recovery actions with the help of
Taskflow persistence feature. Masakari will persist the flow so that it can be
resumed, restarted or rolled-back on engine failure.

Taskflow supports persistence of workflow which helps to persist each task
details in the database. For more details please refer `persistence-doc`_

Taskflow has below three tables where workflow/task details are getting
stored:

* logbooks
* flowdetails
* atomdetails

In particular, for each flow there is a corresponding flowdetails
record, and for each task there is a corresponding atomdetails record. These
form the basic level of information about how a flow will be persisted.

With the help of importing persistence package `taskflow_persistence`_ and by
accessing Masakari storage via masakari engine, able to import Taskflow tables
into Masakari. In taskflow library there is workflow, and each workflow has
task which has state and status. With the help of `notifier_method`_ will
update progress details for detailed execution flow for each task of recovery.

Saved recovery task details (failures, successes, intermediary results) going
to render on Horizon on tabular format which helps operators to understand
progress/status of recovery. Each flow execution details stored with scale
0 to 1, so that operator will able to get progress completion along with
detailed information of each task.

Explaining below the how actions/events that going to be recorded for
‘instance-failure recovery workflow’ along with progress details:

* Stop Instance Task: Below listed are possible events along with progress
  details that will be recorded:

  * Starting of Stop instance task::

      "progress_details" = {
        "progress": 0.50,
        "progress_data": "Started execution of StopInstanceTask <INSTANCE_UUID>"
      }

  * Skipping recovery event if an instance is not HA_Enabled and
    "process_all_instances" config option is also disabled::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Skipping recovery for instance <INSTANCE_UUID> as it is not Ha_Enabled"
      }

  * Ignored recovery event if an instance VM state is either in 'paused',
    'rescued'::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Ignoring recovery for instance <INSTANCE_UUID> as it is in paused/rescued state"
      }

  * Stop instance event::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Finished execution of StopInstanceTask <INSTANCE_UUID>"
      }

  * Failure event in case failed to stop instance::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Failed to stop instance <INSTANCE_UUID>"
      }

* Start Instance Task: Below listed are possible events along with progress
  details that will be recorded:

  * Start instance event::

      "progress_details" = {
        "progress": 0.5,
        "progress_data": "Started execution of StartInstanceTask <INSTANCE_UUID>"
      }

  * Finish of Start instance event::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Finished execution of StartInstanceTask <INSTANCE_UUID>"
      }

  * Failure event in case failed to start instance or if invalid state of it::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Failed to start instance <INSTANCE_UUID>"
      }

* Confirm Instance Active Task: Below listed are possible events along with
  progress details that will be recorded:

  * Start of Confirm instance event::

      "progress_details" = {
        "progress": 0.5,
        "progress_data": "Confirming instance <INSTANCE_UUID> is Active"
      }

  * Finish of Confirm instance started event::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Confirmed instance <INSTANCE_UUID> is Active"
      }

  * Failure event in case failed to confirm instance::

      "progress_details" = {
        "progress": 1,
        "progress_data": "Failed to confirm instance <INSTANCE_UUID>"
      }

.. note::
   Events are emitted only when masakari engine starts processing received
   notifications by executing recovery workflow.

Mentioning below the database entries that going to be recorded for
‘instance-failure recovery workflow’::

    LogBook: 'instance_recovery'
    - uuid = 68e86fda-25ba-4b1d-a9fc-d999bc1c796e
    - created_at = 2019-01-08 08:15:21
    - updated_at = 2019-01-08 08:15:21
    - meta: {"notification_uuid": "9ca38361-eef9-4fca-a1fe-49ef0c7e23e8"}
    FlowDetail: 'instance_recovery_engine'
    - uuid = 6a780ae7-9c63-42d9-8510-aa020d7ee566
    - state = SUCCESS
    TaskDetail: 'StopInstanceTask'
    - uuid = c165b8c2-5123-4489-99c1-97eafff72d24
    - state = SUCCESS
    - version = 1.0
    - failure = False
    - meta: {}
    - results: <CONTEXT_DETAILS>
    TaskDetail: 'StopInstanceTask'
    - uuid = c165b8c2-5123-4489-99c1-97eafff72d24
    - state = SUCCESS
    - version = 1.0
    - failure = False
    - meta:
        + progress = 100.00%
        + progress_details = {
            "progress": 1,
            "progress_details": {
                "at_progress": 1,
                "details": {
                    "progress_details": [
                        "progress_details" = {<progress_details_of_event_1>, <progress_details_of_event_2>, ..., <progress_details_of_event_n>}
                    ]
                }
            }
         }
    - results: NULL
    TaskDetail: 'StartInstanceTask'
    - uuid = a4155556-fb5a-44f8-b8aa-ab8ecfe8f1ce
    - state = SUCCESS
    - version = 1.0
    - failure = False
    - meta:
        + progress = 100.00%
        + progress_details = {
            "progress": 1,
            "progress_details": {
                "at_progress": 1,
                "details": {
                    "progress_details": [
                        "progress_details" = {<progress_details_of_event_1>, <progress_details_of_event_2>, ..., <progress_details_of_event_n>}
                    ]
                }
            }
         }
    - results: NULL
    TaskDetail: 'ConfirmInstanceActiveTask'
    - uuid = 0ea82633-599b-422d-8fd2-df2057efb29d
    - state = SUCCESS
    - version = 1.0
    - failure = False
    - meta:
        + progress = 100.00%
        + progress_details = {
            "progress": 1,
            "progress_details": {
                "at_progress": 1,
                "details": {
                    "progress_details": [
                        "progress_details" = {<progress_details_of_event_1>, <progress_details_of_event_2>, ..., <progress_details_of_event_n>}
                    ]
                }
            }
         }
    - results: NULL


Mentioning below how the recorded data will be used to render task details
in tabular format for ‘instance-failure recovery workflow’ on Horizon::

    * Stop Instance Task
    ============================================  ==========================  ==========================  ====================================================
    Request ID                                    Action                      Start Time                  Message
    ============================================  ==========================  ==========================  ====================================================
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      StopInstanceTask            Jan 10 2019, 10:40 a.m      Started execution of StopInstanceTask <INSTANCE_UUID>
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      StopInstanceTask            Jan 10 2019, 10:41 a.m      Finished execution of StopInstanceTask <INSTANCE_UUID>
    ============================================  ==========================  ==========================  ====================================================

    * Start Instance Task
    ============================================  ==========================  ==========================  ====================================================
    Request ID                                    Action                      Start Time                  Message
    ============================================  ==========================  ==========================  ====================================================
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      StartInstanceTask           Jan 10 2019, 10:41 a.m      Starting instance <INSTANCE_UUID>
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      StartInstanceTask           Jan 10 2019, 10:42 a.m      Started instance <INSTANCE_UUID>
    ============================================  ==========================  ==========================  ====================================================

    * Confirm Instance Active Task
    ============================================  ==========================  ==========================  ====================================================
    Request ID                                    Action                      Start Time                  Message
    ============================================  ==========================  ==========================  ====================================================
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      ConfirmInstanceActiveTask   Jan 10 2019, 10:43 a.m      Confirming instance is Active <INSTANCE_UUID>
    req-679033b7-1755-4929-bf85-eb3bfaef7e0b      ConfirmInstanceActiveTask   Jan 10 2019, 10:43 a.m      Confirmed instance is Active <INSTANCE_UUID>
    ============================================  ==========================  ==========================  ====================================================

Alternatives
------------

Send Versioned notifications similar to the other OpenStack services for
recovery workflows.

Data model impact
-----------------

Below tables will get added into Masakari Database

* alembic_version
* logbooks
* flowdetails
* atomdetails

.. note::
   alembic_version here stores version information of taskflow database
   version, not of Masakari database.
   Masakaari database as of now is not under alembic control.

For example in case of ‘instance-failure recovery workflow’, data will be
stored in below columns

* logbooks: Parent table, one entry for each notification received.
* flowdetails: Child table for logbooks, one entry for each notification received.
* atomdetails: Child table for flowdetails, one entry for each task of recovery.

.. note::
   Foreign key association is not there for taskflow persistence tables.
   If we delete logbook entry, respective child entries also got deleted.

REST API impact
---------------

A new microversion will be created to add event details to GET
/notifications/<notification_uuid> API.

Security impact
---------------

None

Notifications impact
--------------------

Masakari recovery failure doesn't support event notification feature.
This spec will add this feature.

Other end user impact
---------------------

None

Performance Impact
------------------

There will be a slight performance impact due to the overhead for storing
events during processing of each recovery failure into database.

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

* Jayashri Bidwe <Jayashri.Bidwe@nttdata.com>
* Vrushali Kamde <Vrushali.Kamde@nttdata.com>

Work Items
----------

* Fetch backend as Masakari backend for each taskflow
* Execute taskflow with all details at each task that required
* Populate meta with progress status
* Update the notification API for GET /notifications/<notification_uuid> in a
  new microversion to pass the stored event related information of recovery
  failure
* Update unit tests for code coverage
* Add documentation on how to use this feature at Horizon


Dependencies
============

None


Testing
=======

No need to write tempest tests as unit tests are sufficient to check
whether the events are sent or not for recovery operations.


Documentation Impact
====================

None


References
==========

..  _`persistence-doc`: https://docs.openstack.org/taskflow/latest/user/persistence.html
..  _`taskflow_persistence`: https://github.com/openstack/taskflow/tree/master/taskflow/persistence
..  _`notifier_method`: https://github.com/openstack/taskflow/blob/master/taskflow/types/notifier.py#L186


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Stein
     - Introduced
