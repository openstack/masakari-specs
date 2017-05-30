..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

..

==================================
 Introspective Instance Monitoring
==================================

https://blueprints.launchpad.net/masakari/+spec/introspective-instance-monitoring

Currently, Masakari instance monitoring is strictly non-intrusive black-box
type monitoring through qemu and libvirt.  There are however a number of
internal instance/VM faults (kernel scheduling and IO, application health),
that if detected by Masakari, could be recovered by existing Masakari auto-recovery
mechanisms; increasing the overall availability of the instance/VM.  This blueprint
introduces the capability of performing introspective instance monitoring of VMs, in
order to detect, report and optionally recover VMs from internal VM faults.  Specifically,
VM Heartbeat Monitoring via the QEMU Guest Agent is introduced by this spec, in order
to indirectly detect some of these internal VM faults.



Problem description
===================

Currently, Masakari instance monitoring is a strictly non-intrusive black-box
type monitoring through qemu and libvirt.  This detects a number of faults
for which  Masakari's auto-recovery mechanisms can be used to recover the
instance/VM.

However, there are a number of internal instance/VM faults not detected by
this black-box monitoring, that if detected by Masakari, could be recovered
by these same Masakari auto-recovery mechanisms.  This includes faults such as
hung Guest OS, failure of the Guest OS to schedule Application process(es), failure
to route basic IO within the Guest, Application-specific process failures or data
corruption, etc. .  The exact scope of the proposed monitoring of this blueprint
is described at the end of the 'Proposed change' section.

Monitoring of Internal instance/VM faults requires that the Guest VM
supports software to respond to this monitoring.  In the following proposal,
the Guest VM must support the QEMU Guest Agent.  Because not all VMs will support
this software, the monitoring of internal instance/VM faults, by the OpenStack Host,
must be optionally enabled per VM or per VM image.



Proposed change
===============

This blueprint introduces introspective instance monitoring; specifically, VM
Heartbeat Monitoring via the QEMU Guest Agent.  Any VM Heartbeat fault will be
reported through the Masakari instance-alerter to registered  API drivers
(e.g. masakari-api).

The high-level architecture for Introspective Instance Monitoring is shown below::

   +--------------------+   instance  +-------------+    + - - - - - - +
   | instance-alerter   |<------------|  Masakari   |    |             |
   |- - - - - - - - - - |   fault     |     VM      |      F U T U R E
   | driver abstraction |             |  Heartbeat  |    |             |
   |       layer        |             |    Agent    |
   +--------------------+             +-------------+    + - - - - - - +
              |    |                         ^                  ^
     other <--+    |                         |                  |
     apis          |                         | +----------------+
                   v                         | |
   +--------------------+                    | |
   |    masakari-api    |                    v v
   +--------------------+             +-------------+
            |                         |  Libvirtd   |
            v                         +-------------+
   +--------------------+                    ^
   |   masakari-engine  |                    | unix socket
   +--------------------+                    v
            |                         +-------------+
            | (recovery)              |    QEMU     |
            v                         +-------------+
   +--------------------+                    ^
   |                    |                    |
   |      OpenStack     |       +--------------------------------------+
   |                    |       | VM         | virtio serial device    |
   +--------------------+       |            v                         |
                                |       +--------------------+         |
                                |       |   QEMU             |         |
                                |       |   Guest Agent      |         |
                                |       |   ( guest-ping{} ) |         |
                                |       +--------------------+         |
                                |                                      |
                                |         +-------------+              |
                                |       +-------------+ |              |
                                |       |             | |              |
                                |       | Application | |              |
                                |       |             | +              |
                                |       +-------------+                |
                                +--------------------------------------+


VM Heartbeat and Healthcheck Monitoring will leverage the QEMU feature, Guest
Agent [1], for both the transport level
communication between OpenStack Host and the Guest VM, and the built-in
guest ping command (guest-ping{}).  A QEMU Guest Agent
daemon, built as part of QEMU, is installed and run inside the Guest and
implements support for QMP commands that are sent to
the guest.  Specifically the QEMU Guest Agent daemon
connects to a virtio-serial device (/dev/virtio-ports/org.qemu.guest_agent.0),
feeds the input to a QMP JSON parser, and when a command is received, invokes
the QAPI generated dispatch routine.  In the case of VM Heartbeat Monitoring,
the QEMU Guest Agent command, 'guest-ping', will be used as the heartbeat challenge
request from the Host.

On the host, OpenStack Nova already supports an image property,
hw_qemu_guest_agent, that can be used to specify that the VM should
be created with the QEMU guest agent virto-serial-interface.  The Masakari
VM Heartbeat Agent will discover VMs with hw_qemu_guest_agent enabled
by monitoring the files representing the socket identifiers for the QEMU Guest
Agents' virtual-serial-interfaces.

libvirt-qemu provides a virDomainQemuAgentCommand() for sending commands
to a selected VM's QEMU guest agent.  This command opens the unix socket to
the VM's virtio-serial-interface, sends the command, waits to receive the response
and closes the socket.  The command fails if the unix socket is openned by
another process, i.e. another process is sending a command to the same VM.

Masakari VM Heartbeat Agent will leverage virDomainQemuAgentCommand() provided
by libvirtd to send the heartbeat challenge requests (i.e. the QEMU Guest Agent's
guest-ping command) to the VM(s) and report any detected faults to the masakari
instance-alerter.

The Masakari VM Heartbeat Agent, on the host, will initiate VM Heartbeating as soon
as it discovers the VM has QEMU Guest Agent communication enabled.  However, in order
to deal with arbitrary boot times for VMs/Guests, which may delay the Guests ability
to start responding to the heartbeat challenges, the Masakari VM Heartbeat Agent will
not enable reporting of heartbeat failures until after the first successful heartbeat
response is received from the VM/Guest.

This functionality will support a flag in masakari.conf for overall enabling/disabling of
introspective-instance-monitoring.  It will also support parameters for configuring
default heartbeat period and default consecutive heartbeat miss threshold (before
declaring fault); in future, flavor extraspecs could be used for VMs to specify
specific values for these.

At a high-level, the scope of this heartbeat monitoring is that the QEMU Guest Agent
is running within the VM.  However, just the fact that a Heartbeat message can get
from the Host to the QEMU Guest Agent inside the VM and back, inherently validates
that a lot of basic Guest Kernel functionality is working; i.e. the Guest OS is not
hung or failed, the QEMU heartbeat message was properly routed through basic linux
socket IO, etc. .  In the future, the heartbeating can be extended to
do more than just reply/ack the message ... i.e. basic sanity / health tests on key
applications within the VM can be done.




Alternatives
------------

Could simply leverage the virtual hardware watchdog of QEMU/KVM
[2] for Instance monitoring.

However, VM Heartbeat Monitoring:

- provides notification of the Heartbeat status to higher-level cloud
  entities through instance-alerter, such as Masakari, Mistral and/or Vitrage,

   * which depending on the backend can result in VM auto-recovery (Masakari) or
     deduced-state updates in Nova for the VM and resulting Aodh Event generation
     due to the VM state change (Vitrage).

- in the future can be extended to provide a higher-level (i.e. application-level)
  heartbeating

   * i.e. if the Heartbeat requests are being answered by the Application running
     within the VM

- in the future can be extended to provide more than just heartbeating, as the
  Application can use it to trigger a variety of audits,

- in the future can be extended to provide a mechanism for the Application within the
  VM to report a Health Status / Info back to the Host / Cloud.



Limitation
----------

Only VMs supporting the QEMU Guest Agent can be monitored by the functionality of
this proposal.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  greg-waines


Milestones
----------

Target Milestone for completion:
  Rocky-2


Work Items
----------

- Masakari VM Heartbeat Agent on the Compute

   * discovery of VMs with QEMU Guest Agent communication enabled,

   * high-level logic for Heartbeat / Healthcheck monitoring,

   * reporting of faults to masakari instance-alerter.

- tox and/or tempest test suite updates

- masakari documentation updates



Dependencies
============

- requires that VMs are installed with and running the QEMU Guest Agent [1]
  built as part of QEMU.


References
==========

[1] http://wiki.qemu.org/Features/GuestAgent

[2] https://libvirt.org/formatdomain.html#elementsWatchdog

