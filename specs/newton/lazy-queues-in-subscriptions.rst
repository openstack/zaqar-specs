..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

============================
Lazy queues in subscriptions
============================

https://blueprints.launchpad.net/zaqar/+spec/lazy-queues-in-subscriptions

Make queues lazy on operations with subscriptions, so the user will be able to
subscribe to yet unexisting queue.

Problem description
===================

Queues are designed to be lazy resources in Zaqar's API v1.1 and API v2 in
messaging. That means, for example, that the user can post messages to
unexisting queue, and the queue will be automatically created. But now queues
do not behave like lazy resources on operations with subscriptions. Before
creating a subscription, the user must ensure the queue exists, or Zaqar will
send error response with HTTP code 400.

Proposed change
===============

Make queues lazy on operations with subscriptions, so the user will be able
to subscribe to yet unexisting queue. Make Zaqar do not send error response
on queue creation operation in the case queue does not exists.

Advantages:

#. The behavior of API will be more consistent, because after the change
   queues will be lazy in all situations.
#. It will be easier for the user to work with subscriptions. No need to do
   pre-check for queue existence, or handle 400 error response from Zaqar, or
   always pre-create queue.
#. There will be no need to solve currently existing problem when the queue was
   deleted, but subscriptions to it are still existing. Problem solving will
   probably require storage driver code to delete also all subscriptions to
   a queue, when deleting the queue. It may be very undesirable for the user,
   because subscriptions will not guarantee to be existing till TTL expiration,
   and will need to be renewed also after queue delete and queue create
   operations. Proper solution to this problem will complicate behavior in many
   ways in both Zaqar and client's code. It will be probably backward
   incompatible change.

The change is backward compatible. Client's code should work after the change.

Drawbacks
---------

None

Alternatives
------------

Two alternatives:

#. Keep the current behavior.
#. There is opinion to make queues non-lazy resources in all situations like
   it was in API v1. But it will be backward incompatible change to our API
   v1.1 and API v2, so maybe make it in the distant future.

Both of them will require solving the problem about still active subscriptions
to a queue that was deleted.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ubershy

Milestones
----------

Target Milestone for completion:
  Newton-1

Work Items
----------

#. Remove checks for queue existence on operations with subscriptions from
   storage drivers.
#. Remove the code that is catching ``QueueDoesNotExist`` exception on
   operations with subscriptions from transport drivers.
#. Change functional and unit tests accordingly.

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode
