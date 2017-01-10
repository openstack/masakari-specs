===============================
README
===============================

OpenStack Masakari Specifications
=============================


This git repository is used to hold approved design specifications for additions
to the Masakari project. Reviews of the specs are done in gerrit, using a
similar workflow to how we review and merge changes to the code itself.

The layout of this repository is::

  specs/<release>/

Where there are two sub-directories:

  specs/<release>/approved: specifications approved but not yet implemented
  specs/<release>/implemented: implemented specifications


The lifecycle of a specification
--------------------------------

Developers proposing a specification should propose a new file in the
``approved`` directory. masakari-core will review the change in the usual
manner for the OpenStack project, and eventually it will get merged if a
consensus is reached. At this time the Launchpad blueprint is also approved.
The developer is then free to propose code reviews to implement their
specification. These reviews should be sure to reference the Launchpad
blueprint in their commit message for tracking purposes.

Once all code for the feature is merged into Masakari,
the Launchpad blueprint is marked complete.
As the developer of an approved specification it is your
responsibility to mark your blueprint complete when all of the required
patches have merged.

Periodically, someone from masakari-core will move implemented specifications
from the ``approved`` directory to the ``implemented`` directory.
Individual developers are also welcome to propose this move for their
implemented specifications.
It is important to create redirects when this is done so that
existing links to the approved specification are not broken. Redirects aren't
symbolic links, they are defined in a file which sphinx consumes. An example
is at ``specs/ocata/redirects``.

This directory structure allows you to see what we thought about doing,
decided to do, and actually got done. Users interested in functionality in a
given release should only refer to the ``implemented`` directory.


Example specifications
----------------------

You can find an example spec in ``specs/ocata-template.rst``.


Working with gerrit and specification unit tests
------------------------------------------------

For more information about working with gerrit, see
http://docs.openstack.org/infra/manual/developers.html#development-workflow

To validate that the specification is syntactically correct (i.e. get more
confidence in the Jenkins result), please execute the following command::

  $ tox

After running ``tox``, the documentation will be available for viewing in HTML
format in the ``doc/build/`` directory.


* Free software: Apache license
* Documentation: http://docs.openstack.org/developer/masakari-specs

* TODO
