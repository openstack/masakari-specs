..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Recovery method customization
=============================

https://blueprints.launchpad.net/masakari/+spec/recovery-method-customization

This spec talks about making recovery workflow configurable. Operator can
configure the workflow in a config file which can be used to build and
execute the recovery workflow.

What is recovery workflow?

Recovery workflow is nothing but certain set of actions executed to recover
from failure.
Masakari supports three types of recovery failures:

* instance-failure
* process-failure
* host-failure

For each of these failures, Masakari executes a workflow to recover from
failure on receiving the notification.

Problem description
===================

Masakari uses taskflow library to execute the workflows which consists of
recovery actions which are predefined and are executed linearly. If operator
wants to add/remove any existing recovery actions to any of these workflow,
then there is no other way to do so without making changes in the code.
For example in case of ‘host-failure recovery workflow’, predefined workflow is:

* disable_compute_node
* prepare_ha_enabled_instances
* evacuate and confirm_evacuate

If operator wants to remove task ‘disable_compute_node’ from the workflow or add
a new task such as send an alert mail to operator then it's not possible with
the current implementation.

Use Cases
---------

Operator may want to add/remove tasks from the existing workflow based on
their requirements.
For example, in case of ‘host-failure recovery’ workflow predefined flow is;
disable_compute_node, prepare_ha_enabled_instances, evacuate and
confirm_evacuate.
Some of the possible recovery workflow combinations can be:

* send Alert/Mail to operator/users of vms -> disable_compute_node
  -> prepare_ha_enabled_instances -> evacuate
  -> update the pricing/ metering DB -> confirm_evacuate
  -> send Alert/Mail to operator/user (recovery done)
* send Alert/Mail to operator/users of vms -> disable_compute_node
  -> prepare_ha_enabled_instances
  -> evacuate -> confirm_evacuate
  -> send Alert/Mail to operator/users of vms (recovery done)
* send Alert/Mail to operator/users of vms

Proposed change
===============

Make a provision to add/remove tasks from the existing workflow based on the
requirements. We plan to decompose the existing hard-coded recovery workflow
into separate tasks and then tied them together to form a workflow which can be
configured in a new conf ‘masakari-custom-recovery-methods.conf’ file as explained
below:
Add a section ‘[taskflow_driver_recovery_flows]’ in newly added
masakari-custom-recovery-methods.conf file. Under this add below config options for
configuration of customized recovery actions for each type of workflow.
Each config option will be dictionary containing key/value pairs for
tasks to be executed.
For example: pre:[v1,v2],main:[v1,v2],post:[v1,v2,v3]
Here key will be pre/main/post and value will be the list of tasks to execute
for recovery failure.
If file does not exist, then default tasks will be executed that will be
configured during registration of configuration options.

* ‘instance_failure_recovery_tasks’ is a dictionary containing key as
   pre/main/post and value will be the comma-separated list of tasks to be
   executed for process failure.
* ‘process_failure_recovery_tasks’ is a dictionary containing key as
   pre/main/post and value will be the comma-separated list of tasks to be
   executed for process failure.
* ‘host_auto_failure_recovery_tasks’ is a dictionary containing key as
   pre/main/post and value will be the comma-separated list of tasks to be
   executed for host failure for auto recovery.
* ‘host_rh_failure_recovery_tasks’ is a dictionary containing key as
   pre/main/post and value will be the comma-separated list of tasks to be
   executed for host failure for rh recovery.

For example,

.. code::

    [taskflow_driver_recovery_flows]
    instance_failure_recovery_tasks = pre:['custom_pre_task','stop_instance_task'],
                                      main:['start_instance_task','custom_main_task'],
                                      post:['confirm_instance_active_task','custom_post_task']
    process_failure_recovery_tasks = pre:['disable_compute_node_task'],
                                     main:['confirm_compute_node_disabled_task','custom_main_task'],
                                     post:['custom_post_task']
    host_auto_failure_recovery_tasks = pre:['disable_compute_service_task'],
                                       main:['prepare_HA_enabled_instances_task'],
                                       post:['evacuate_instances_task','custom_post_task']
    host_rh_failure_recovery_tasks = pre:['custom_pre_task','disable_compute_service_task'],
                                     main:['prepare_HA_enabled_instances_task',
                                     'evacuate_instances_task']
                                     post:['custom_post_task']

Need to add entry point for each task in setup.cfg so that these tasks can be
loaded dynamically using stevedore during creation of a recovery workflow.

For example, Masakari setup.cfg will have following entry points:

* For each entry point in setup.cfg should have the full class path as mentioned
  in below example:

.. code::

    masakari.task_flow.tasks =
        stop_instance_task = masakari.engine.drivers.taskflow.instance_failure:StopInstanceTask

.. code::

    masakari.task_flow.tasks =
        disable_compute_service_task = <full_class_path_of_task>
        prepare_HA_enabled_instances_task = <full_class_path_of_task>
        evacuate_instances_task = <full_class_path_of_task>
        stop_instance_task = <full_class_path_of_task>
        start_instance_task = <full_class_path_of_task>
        confirm_instance_active_task = <full_class_path_of_task>
        disable_compute_node_task = <full_class_path_of_task>
        confirm_compute_node_disabled_task = <full_class_path_of_task>

If operator wants to configure customized tasks in a Third Party library,
then they will need to follow below guidelines to associate newly added
tasks with the respective recovery workflows in Masakari:

* First make sure required Third Party Library is installed on the Masakari
  engine node.
* Configure custom task in Third Party Library's setup.cfg as below:

For example, Third Party Libraries setup.cfg will have following entry points

.. code::

    masakari.task_flow.tasks =
        custom_pre_task = <custom_task_class_path_from_third_party_library>
        custom_main_task = <custom_task_class_path_from_third_party_library>
        custom_post_task = <custom_task_class_path_from_third_party_library>

Note:
    Entry point in Third Party Library's setup.cfg should have same key as
    in Masakari setup.cfg for respective failure recovery.

* If there are any configuration parameters required for custom task,
  then add them into masakari-custom-recovery-methods.conf under the
  same group/section where they are registered in Third Party Library.
  Operator will be responsible to generate masakari configuration file
  by themselves.

* Operator should ensure output of each task should be made available to
  the next tasks needing them.


Alternatives
------------

For recovery from failures, instead of fully configurable task flow,
one can add custom tasks at the start or after completion of predefined
existing workflow.

One can customized recovery workflow in masakari-custom-recovery-methods.conf
as below and Masakari will inject these custom tasks at start or end of the
predefined workflow as per requirement.

For example,

.. code::

    [taskflow_driver_recovery_flows]
    instance_failure_recovery_tasks = ['custom_pre_task','custom_main_task']
    process_failure_recovery_tasks = ['custom_pre_task']
    host_auto_failure_recovery_tasks = ['custom_pre_task','custom_main_task']
    host_rh_failure_recovery_tasks = ['custom_pre_task']


custom_pre_task and custom_main_task will be executed at the start or end of
the existing ‘instance_failure’ workflow.

Note:
    For host failure having recovery method as rh, developer should add
    custom task in nested flow so that it will execute once.

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

A new config file ‘masakari-custom-recovery-methods.conf’ will be added, where
‘taskflow_driver_recovery_flows’ section need to be update for customized
recovery workflows.
If an operator doesn't want any customization to any of the recovery workflows,
then there will be no impact as it will load the default tasks for each
recovery workflow.
For example,

.. code::

    [taskflow_driver_recovery_flows]
    instance_failure_recovery_tasks = pre:['custom_pre_task','stop_instance_task'],
                                      main:['start_instance_task','custom_main_task'],
                                      post:['confirm_instance_active_task','custom_post_task']
    process_failure_recovery_tasks = pre:['disable_compute_node_task'],
                                     main:['confirm_compute_node_disabled_task','custom_main_task'],
                                     post:['custom_post_task']
    host_auto_failure_recovery_tasks = pre:['disable_compute_service_task'],
                                       main:['prepare_HA_enabled_instances_task'],
                                       post:['evacuate_instances_task','custom_post_task']
    host_rh_failure_recovery_tasks = pre:['custom_pre_task','disable_compute_service_task'],
                                     main:['prepare_HA_enabled_instances_task',
                                     'evacuate_instances_task']
                                     post:['custom_post_task']

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:

* bhagyashris <bhagyashri.shewale@nttdata.com>

Work Items
----------

* Implement customize task flow execution.
* Add unit tests for the coverage.
* Add documentation guide to describe how to configure customizable workflows.

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Add documentation guide to describe how to configure customizable workflows.

References
==========

https://etherpad.openstack.org/p/masakari-recovery-method-customization
http://eavesdrop.openstack.org/meetings/masakari/2018/masakari.2018-07-03-03.00.log.html

History
=======

None
