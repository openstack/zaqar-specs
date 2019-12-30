..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=======================
Query Queues With Count
=======================

https://blueprints.launchpad.net/zaqar/+spec/query-queues-with-count

This will support query queues with 'with_count=true' filter, Zaqar will return
the amount of queues in backend storage. This feature will be very convenient
to users to know how many resources they own.

Problem description
===================

Currently, Zaqar can't return the amount of queues when querying the queue.
It depends on users themselves to calculate the number one by one. For other
clients or applications also need to do it after invoking Zaqar's API. Its
quite inconvenient for users or developers.

Proposed change
===============

Add a new query filter item named ``with_count``, default value is ``False``.
When querying queues with "with_count=true" in url, Zaqar will add the
function to calculate total number of queus in backend storage and
return the amount of queues in response body like "count=100".

API Impact
-----------
Query queue list
GET: /v2/queues?with_count=true

  RESPONSE CODE: 200
  RESPONSE BODY:
  {
    "count": 100,
    "queues": [...]
  }

Drawbacks
---------

None

Alternatives
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao <sxmatch1986@gmail.com>

Secondary assignee:
  None

Milestones
----------

Target Milestone for completion:
  ussuri RC2

Work Items
----------

#. Modify transport code.
#. Add release note for this feature.
#. Update API reference.
#. Change unit, functional and tempest tests accordingly.
#. Add client support.

Dependencies
============

None

References
==========

None