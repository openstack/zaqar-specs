..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=============================
Delete Message With Claim ID
=============================

https://blueprints.launchpad.net/zaqar/+spec/delete-message-with-claim-id

Delete Message with claim id means that when a user deletes a message, the message
must be claimed. If you want to delete a message, you will have to use both message
id and claim id. This can improve the security of the message.

Problem description
===================

Currently, any client who knows the message ID can delete the message if it not be
claimed. It could cause some unexpected problems. A better way to delete a message
is make sure the message is deleted by the client who is claiming the message.
Amazon SQS use receipt handler to delete a message[1]. Zaqar can use claim id and
message id to delete messages.

Proposed change
===============

Add a new configuration item named ``message_delete_with_claim_id``, default value
is ``False``, means it is backwards compatible. You can modify this configuration
item to decide whether to turn on the switch. If you change it to ``True``, you
need to forcibly carry the claim id when delete messages. If the claim ID is invalid,
the message can not be deleted. You must re-claim the messages, and then delete it.

..note::

   No matter "message_delete_with_claim_id" is True of False, admin can
   always delete a message without claim_id.

API Impact
-----------
Delete single message
DELETE: /v2/queues/test_queue/messages/{message_id}?claim_id={claim_id}

  RESPONSE CODE: 204

Delete messages
DELETE: /v2/queues/test_queue/messages?ids={messages_ids}&claim_ids={claim_ids}

  RESPONSE CODE: 204


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
  cdyangzhenyu <cdyangzhenyu@gmail.com>

Secondary assignee:
  gecong<ge.cong@zte.com.cn>
  wanghao<sxmatch1986@gmail.com>

Milestones
----------

Target Milestone for completion:
  stein RC3

Work Items
----------

#. Modify message delete code.
#. Add release note for this feature.
#. Update API reference.
#. Add user/developer document for this feature.
#. Change unit, functional and tempest tests accordingly.

Dependencies
============

None

References
==========

[1]:https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-message-identifiers.html