==================
Masakari Spec Lite
==================

Please keep this template section in place and add your own copy of it between
the markers. Please fill only one Spec Lite per commit.

<Title of your Spec Lite>
-------------------------

:link: <Link to the blueprint.>

:problem: <What is the driver to make the change.>

:solution: <High level description what needs to get done. For example:
            "We need to add client function X.Y.Z to interact with new server
            functionality Z".>

:impacts: <All possible \*Impact flags (same as in commit messages) or 'None'.>

Optionals (please remove this line and fill or remove the rest until End):
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

:how: <More technical details than the high level overview of `solution`
       if needed.>

:alternatives: <Any alternative approaches that might be worth of bringing
                to discussion.>

:timeline: <Estimation of the time needed to complete the work.>

:reviewers: <If reviewers has been agreed for the functionality, list them
             here.>

:assignee: <If known, list who is going to work on the feature implementation
            here>

End of Template
+++++++++++++++

Add periodic task to clean up workflow failure
----------------------------------------------

:link: https://blueprints.launchpad.net/masakari/+spec/add-periodic-tasks

:problem: Due to some unknown circumstances, there is a possibility few of the
          notifications might get into ‘error’ status or it might remain in
          ‘new’ status forever. There should be some way to retrieve such
          notifications and process them to completion.

:solution: Add a periodic task “process_unfinished_notifications’ which will
           execute at regular interval as defined by the new config option
           “process_unfinished_notifications_interval” in seconds. Default
           value for this option will be 120 seconds, however operator can set
           this interval value as per the requirement. Inside this
           periodic task, it will retrieve all notifications which are in
           ‘error’ or ‘new’ status and then execute recovery action workflow to
           process all of them. The notifications which are in ‘new’ status
           will be picked up based on a new config option
           ‘retry_notification_new_status_interval’. Default value for this
           option will be 60 seconds, however operator can set this interval
           value as per the requirement. Each notification has ‘generated_time’
           field, if this time + retry_notification_new_status_interval value
           (in seconds) is greater than or equal to the current system time,
           then all such notifications in ‘new’ status will be picked up by
           this periodic task. Also, the notifications in ‘error’ status will
           be picked up too.

           Let’s understand the transition state of notification for different
           statuses for success case:
           notification current status error -> running -> finished
           notification current status new-> running -> finished
           If the workflow execution fails, then the transition state of
           notification would be:
           notification current status error -> running -> failed
           notification current status new-> running -> failed

           Note: One important point to take note of is if the original
           notification status is ‘new’ then it won’t be retried again if the
           workflow fails to process it in the periodic task. It’s status will
           be directly set to ‘failed’. The operator needs to take corrective
           action for all notifications which are in ‘failed‘ state manually.

:alternatives: Add two periodic tasks ‘process_error_notifications’ and
               ‘process_queued_notifications’ to process notifications which
               are in ‘error’ and ‘new’ status respectively. These periodic
               tasks will be called at regular intervals as defined by two
               new config options “process_error_notification_interval’ and
               ‘process_queued_notifications_interval’. The logic for retrieval
               of notifications which are in ‘error’ and ‘new’ statuses will be
               exactly same as above solution. The only difference would be
               in the notification status upon its completion as explained
               below.

               Transition state of notification for different statuses for
               success case.
               notification current status
               error (process_error_notifications) -> running -> finished
               notification current status
               new (process_queued_notifications) -> running -> finished

               If the workflow execution fails, then the transition state of
               notification would be:
               notification current status
               error (process_error_notifications) -> running -> failed
               notification current status
               new (process_queued_notifications) -> running -> error
               This means that the notification will be again eligible for
               reprocessing during the next cycle of
               ‘process_error_notifications’ periodic task.

:impacts: None

:timeline: Expected to be merged within the Ocata time frame.

:reviewers: sam47priya@gmail.com, kajinamit@nttdata.co.jp,
            tushar.vitthal.patil@gmail.com

:assignee: Abhishek Kekane

End of Add periodic task to clean up workflow failure
+++++++++++++++++++++++++++++++++++++++++++++++++++++
