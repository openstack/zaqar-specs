..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===========
Purge queue
===========

https://blueprints.launchpad.net/zaqar/+spec/purge-queue

Though user can delete messages, claims or subscriptions from a queue, but
there is no way to delete many of them at once.

Problem description
===================

Now Zaqar is missing a fast way to delete many resources at once for a given
queue. This could be handy for user to clean the queue and keep all the
metadata of the queue.

Proposed change
===============

New endpoint for action 'purge' for queue:

/v2/queues/myqueue/purge

POST body:

.. code:: json

  {"resource_types": ["messages", "subscriptions"]}

So the idea is if there is no POST body, by default, all the resources under
the queue will be delete. Otherwise, if the key 'resource_types' in the POST
body, then Zaqar will delete resources based on the given resource types. It's
a list and it could be one of the combinations of the two types: 'messages',
and 'subscriptions'.

Currently, it's hard to list "claims" under a queue, so this feature won't
support clean "claims".

Drawbacks
---------

N/A

Alternatives
------------

User has to delete messages and subscriptions of the queue manually.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flwang (flwang@catalyst.net.nz)

Work Items
----------

1. Add a new method 'purge' for storage queue controller which will delete all
   messages and subscriptions under the queue.


Dependencies
============

N/A

