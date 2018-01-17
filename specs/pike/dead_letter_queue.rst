..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=================
Dead Letter Queue
=================

https://blueprints.launchpad.net/zaqar/+spec/dead-letter-queue


Problem description
===================

Currently, Zaqar can't do anything if the message has been claimed many times
and still can't be processed successfully. That could be nice if Zaqar can
provide the dead letter queue support so that user can provide an existing
queue A for queue B's dead letter queue. Dead letter queue storing these
messages allows developers to look for common patterns and potential software
problems.

Proposed change
===============

1. Add three new reserved attributes for queue's metadata: ``_max_claim_count``
and ``_dead_letter_queue``.

``_max_claim_count`` is the max number the message can be claimed. Generally,
it means the message cannot be processed successfully. There is no default
value for this attribute. If it's not set, then that means this feature won't
be enabled for current queue.

``_dead_letter_queue`` is the target the message will be moved to when the
message can't processed successfully after meet the max claim count. It's not
supported to add queue C as the dead letter queue for queue B where queue B has
been set as a dead letter queue for queue A. It's not the case we want to
address in this spec. Technically, even though user can set a dead letter queue
C for queue B(when queue B is a dead letter queue of queue A), because the
moving is triggered by claim, so that means if user won't explicitly claim
messages in queue B, then nothing happened.

``_dead_letter_queue_messages_ttl`` is the new TTL setting for messages when
moved to DLQ. If it's not set, current TTL will be kept.

2. Make sure the DLQ and current queue are in the same pool when doing
validation to get better performance.

3. Add a new filed for message in database which can be used to save the claim
count.

4. Changes for claim create method

   4.1. get current claim count from the message, update the claim count by
   plus one.

   4.2. check if the claim count has exceeded the max claim count defined in
   queue's metadata. If it doesn't exceed the ``_max_claim_count``, just do
   normal claim.

   4.3. If the claim count does exceed the ``_max_claim_count``, move the
   message to the dead letter queue directly. Given the goal of DLQ, we'd like
   to keep the claim information of the message. So depends on the storage
   backend, the "move" could introduce a limitation. For example, for MongoDB
   backend, we may have to move the message from the source queue to DLQ by
   changing the queue name directly. Which means the two have to be created on
   same storage pool.

   After moved the message, Zaqar will do nothing against those
   messages in dead letter queue. User could/will use these message to debug
   why those messages can't be processed successfully. In other words, allows
   developers to look for common patterns and potential software problems from
   those messages.

   4.4. messages moved to DLQ will be updated their TTL based on the setting
   ``_dead_letter_queue_messages_ttl``. If it's not set, currentl TTL of
   messages will be kept.


Drawbacks
---------

Based on current design, we may have a limitation because of the technical
trade off. When move the message from source queue to dead letter queue, it'd
be nice to keep the claim info, but if we want to do that, we have to make sure
the two queues are on same storage pool. The implementation could be different
on different back ends.

Alternatives
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flwang <flwang@catalyst.net.nz>

Milestones
----------

Target Milestone for completion:
  Pike-3

Work Items
----------

#. Add _max_claim_count and _dead_letter_queue as reserved attributes in queue
   metadata
#. Add validation for the new reserved attributes
#. Add a new filed for message to count the claim, this needs to be done
   against maongoDB, swift and redis.
#. Add change in claim create which will add the claim count for messages and
   check if the count has exceed the max claim count defined in queue's
   metadata. If it has exceeded, move the message to the dead letter queue.
#. Add release note for this feature
#. Update API reference
#. Add user/developer document for this feature
#. Change unit, functional and tempest tests accordingly.

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode
