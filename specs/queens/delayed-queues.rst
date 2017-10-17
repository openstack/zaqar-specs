..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

======================================
Delayed Queues
======================================

https://blueprints.launchpad.net/zaqar/+spec/delayed-queues

Delay queues can postpone the delivery of new messages in a queue for a
specific number of seconds. Which is useful for an auditing observer process
the auditor would be guaranteed to see messages before they were claimed,
and could even edit messages if needed.

Problem description
===================

Now one of the big function gaps of Zaqar is the delayed queue. Currently, all
the message posted to the queue will be visible immediately. That's enough for
most of the user cases. However, for some user case, user want the message to
be unavailable to end users for a specific period of time.

Proposed change
===============

Generally, a new attribute of queue named ``_default_message_delay``.
If the attribute value greater than 0, then queue is a delayed queue,
otherwise, it's a normal queue.

All messages sent to the delayed queue can not be immediately claimed and
must wait for a ``delay``. The ``delay`` can be set or update as a metadata
when the queue is created.

Then, when user get or claim messages on a delayed queue, Zaqar will take
``delay`` as a query condition, which is as below:

 Message Create + Delay < Current Time

Basically, there are 3 scenarios:

1. Normal message sent to delayed queue

   If a normal message without ``delay`` attribute sent to a delayed queue,
   then the message will use the queue's default delay ``_default_message_delay``.

2. Message with attribute ``delay`` sent to delayed queue

   If message with ``delay`` attribute sent to a delayed queue, then Zaqar will
   use the message's delay seconds value instead of the delayed queue's delay
   seconds value.

3. Message sent to normal queue

   A normal queue is a queue with the ``_default_message_delay`` value of 0.
   Whether the message is a normal message or a message with a ``delay``
   attribute, it will not have a delay feature when it is sent to the normal queue.
   Zaqar does not perform conditional filtering on the ``delay`` attribute
   when get/claim messages. This means the delay feature does not affect the
   performance of normal queue.

Specific implementation steps:

1. Add a new attribute ``_default_message_delay`` for all queue's metadata.

  The attribute ``_default_message_delay`` default value is 0 means
  it is a normal queue. Its ranges from 0 to ``max_message_delay``
  seconds, the ``max_message_delay`` is configurable and the default
  value is 900 seconds (15 mins).

2. Add a new attribute just for message's request body: ``delay``.

  When post messages to queue, user can add ``delay`` like ``ttl``.
  The message's delay time has a higher priority than queues. If
  message ``ttl < delay``, the message will never be claimed, because
  it is in a delay period and the message's ``ttl`` has expired, so the
  message will be deleted by backend.

3. Add a new field ``d`` in message backend.

  The field ``d`` is a delay expires timestamp. It can be saved
  to the backend storage. When the timestamp is less than or equal to
  the current time, the delay expires.

  ``d`` = ``delay`` + ``MESSAGE_CREATED_TIMESTAMP``

4. Make sure the message can not be claimed until the delay is expired.

5. If the queue is a normal queue means the 'delayed queues' feature wasn't
   being used, it is necessary to ensure that it is close to zero overhead.

6. Support for mongo, redis and swift.

Drawbacks
---------

This may introduce a little bit performance impact, because this needs another
new condition for current messages query. But it will not affect the normal
queue too much.

Alternatives
------------

There is no good option for this feature. User have to wait enough seconds to
get / claim messages against Zaqar.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  cdyangzhenyu <cdyangzhenyu@gmail.com>

Milestones
----------

Target Milestone for completion:
  Queens Q-2

Work Items
----------

#. Add mongodb support.
#. Add redis support.
#. Add swift support.
#. Add release note for this feature.
#. Update API reference.
#. Add user/developer document for this feature.
#. Change unit, functional and tempest tests accordingly.

Dependencies
============

None
