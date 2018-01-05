..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

================================
Support Checksum Of Message Body
================================

https://blueprints.launchpad.net/zaqar/+spec/support-md5-of-body

Now, Zaqar will add a non-URL-encoded message body checksum function.
This is essential for the security of the message body, it can prevent the
message from being tampered.

Problem description
===================

Currently, Zaqar has no checksum of the non-URL-encoded message body. It is
a good feature for the security of the message body.

AWS SQS service has supported the function, that would be a good reference for
Zaqar[1].

Proposed change
===============

When user claim or get the messages, the checksum will be returned with message
body together, So user can use it to verify that the message body is correct.
Currently zaqar only support the checksum of the non-URL-encoded message body based
on MD5 digests. We may support other algorithms in future versions, including SHA1,
sha256, sha512, and so on. So, It is necessary to change the following:

1. Add new property named ``checksum`` for message. This property is a string type.

   When you send a message to queue, Zaqar will use the default checksum algorithm
   MD5 to calculate the ``checksum`` value of the non-URL-encoded message body.
   Finally Zaqar will store the value on the backend. Now the backend Mongodb,
   Redis and Swift can be supported.

2. Return the property ``checksum`` for message when user claims or gets the message.

   When user gets or claims the message, The API will return the k-v pair of ``checksum``
   in the body of the message. The user can then use this value to verify that the
   body of the newly retrieved message is correct or not.

3. This feature is backward compatible.

   For old messages that do not have the checksum attribute, the checksum will return
   ``None`` for both ``claim`` and ``get`` operation. This feature will also transit
   smoothly as the old messages gradually expire.

4. Add new data model for message body and queue on different backend.

   MESSAGE::

        +----------------------+---------+
        |  Name                |  Field  |
        +======================+=========+
        |  checksum            |   cs   |
        +----------------------+---------+
        |  body                |   b     |
        +----------------------+---------+
        |  ...                 |   ...   |
        +----------------------+---------+

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

Milestones
----------

Target Milestone for completion:
  Queens RC1

Work Items
----------

#. Add message body checksum support for Mongodb, Redis and Swift.
#. Add release note for this feature.
#. Update API reference.
#. Add user/developer document for this feature.
#. Change unit, functional and tempest tests accordingly.

Dependencies
============

None

References
==========

[1]:https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_Message.html