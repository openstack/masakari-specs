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

Auto compute node registration
------------------------------

:link: https://blueprints.launchpad.net/masakari-monitors/+spec/auto-compute-node-registration

:problem: If an user runs hostmonitor/instancemonitor/processmonitor without
          registering the hosts in a segment and they detect a failure,
          masakari doesn't perform the recovery process since source host is
          unknown. Therefore, it will be more convenient if these monitors can
          register a host automatically on startup.

:solution: Hostmonitor/Instancemonitor/Processmonitor will register a host in
           a particular segment if not already done on startup of these
           services. The name of the host will be pickup automatically where
           these monitors are running and it will call masakari create host
           API to add a host in a particular segment. The segment to which the
           host will be registered will be configurable. Also, a new config
           option will be introduced to decide whether to register a host
           automatically or an operator will configure it manually outside the
           scope of monitors.

:impacts: None

:timeline: Expected to be merged within the Queens time frame.

:reviewers: sam47priya@gmail.com, honjo.rikimaru@po.ntt-tx.co.jp

:assignee: Kengo Takahara

Auto compute node registration
++++++++++++++++++++++++++++++
