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

Add db purge support
--------------------

:link: https://blueprints.launchpad.net/masakari/+spec/db-purge-support

:problem: Masakari do not have any way to delete notification records from
          notification database table. If there are large number of
          notifications in the db, it's going to slow down db search.
          Similarly we can purge records from other database tables.

:solution: Add db purge support to masakari-manage command which will purge
           the deleted/unused records from the database. As masakari do not
           have delete notification api, notification records will be purged
           on the basis of status and updated_at column. Notifications which
           are in finished, ignored and failed status will be purged from the
           notification table. For other tables records which are marked as
           deleted will be purged from the tables. Two optional command line
           configuration options 'age_in_days' defaults to 30 and 'max_rows'
           defaults to -1 will be introduced to provide operator flexibility
           in purging the records. -1 means purge command will remove all
           records from tables which matches 'age_in_days' criteria. If
           operator specifies 'max_rows' greater than 0 then those many records
           will be purged from entire database (not per table) in one
           operation.

           For notification table as we do not have deleted_at value;
           'age_in_days' will be calculated on the basis of updated_at value.
           For example, if 'age_in_days' is mentioned as 10 while purging then
           notifications which are having status finished,  ignored and failed
           and updated before 10 days will be eligible for purging. For other
           tables, records which were deleted before 10 days will be eligible
           for purging.

           Example:
           $ masakari-manage db purge
           This will purge all records from each table which are deleted or
           updated before 30 days.

           $ masakari-manage db purge --age_in_days 60
           This will purge all records from each table which are deleted or
           updated before 60 days.

           $ masakari-manage db purge --max-rows 50
           This will purge  total 50 records from entire database which are
           deleted  or updated before 30 days.

           $ masakari-manage db purge --age_in_days 60 --max-rows 50
           This will purge 50 records from entire database which are deleted or
           updated before 60 days.

:alternatives: Add new delete_all api which will have status and age_in_days as
               input parameters to delete the notifications. In this case user
               can specify input notification status as either ignored, failed
               or finished and based on age_in_days value those records will
               be deleted from the notification table. For example if user
               specifies status as 'finished' and age_in_days as 10 then
               notifications which are having 'finished' status and updated
               before 10 days will be deleted from the notification table.

               Advantages:
               1. Operator has flexibility to decide which notifications needs
               to be deleted from the notification table.

               Disadvantages:
               1. Only notification records will be deleted in this case,
               records from other tables will remain as it is.

:impacts: None

:timeline: Expected to be merged within the Pike time frame.

:reviewers: sam47priya@gmail.com, sagaray@nttdata.co.jp,
            tushar.vitthal.patil@gmail.com

:assignee: Pooja Jadhav

Add db purge support
++++++++++++++++++++
