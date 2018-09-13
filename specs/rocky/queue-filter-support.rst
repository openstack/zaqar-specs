..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=====================
Queue filter support
=====================

https://blueprints.launchpad.net/zaqar/+spec/queue-filter-support

Metadata is a group of key-value pairs that belong to a queue. When a queue is
created, the user who creates the queue can add the metadata {"keyx": "valuex"}
to distinguish it. It is necessary for zaqar to support the function of queue
filter so that Users can select the queue by the specified key-value of
metadata. In this blueprints, we will also add queue filter by name.


Problem description
===================

Zaqar doesn't support queue filter when queue listing,
we may add this feature now.


Proposed change
===============
When queue listing, we add filter option in query string parameters
like this ::

  GET /v2/queues?key1=value1&key2=value2&name=value3

If metadata of queue and name is consistent with the filter, the queues will be
listed to the user, otherwise the queue will be filtered.
If filter option is enabled, the metadata of the queue will be returned
to user.

API Impact
-----------
Lists queues::

  GET: /v2/queues?key1=value1&key2=value2

  RESPONSE CODE: 200

Drawbacks
---------

N/A

Alternatives
------------

N/A


Implementation
==============

Assignee(s)
-----------

gecong<ge.cong@zte.com.cn>

Milestones
----------

Rocky-1

Work Items
----------

#* Add filter parameter for queues listing REST API.
#* Update API reference.
#* Add release note for this feature.
#* UTs for this feature.

Dependencies
============

N/A

