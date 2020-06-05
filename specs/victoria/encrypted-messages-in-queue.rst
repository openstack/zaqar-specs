..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===========================
Encrypted Messages in Queue
===========================

https://blueprints.launchpad.net/zaqar/+spec/encrypted-messages-in-queue

The queue in Zaqar will support to encrypt messages before storing them into
storage backends, also could support to decrypt (or not) messages when those
are claimed by consumer. This feature will enhance the security of messaging
service.

Problem description
===================

Currently, Zaqar can't encrypt any messages and just store those messages into
storage backends. That'll bring some security issues like information leakage
or hacker attack.

Proposed change
===============

1. Add one new metadata in queue object that will indicate how to encrypt messages:

#. "_enable_encrypt_messages=true/false" : this will tell Zaqar whether encrypt
   messages before storing them into backends or not.

2. Support to use algorithms to encrypt messages in transport layer before
   storing them into backends. In V cycle, Zaqar may just support the global
   key for encryption and decryption, and it will be held by Zaqar service.
   Users don't need to encrypt/decypt message by themselves.

3. Go a step further, Zaqar also could support to let user upload their public
   key and hold the private key by themselves. In this case, Zaqar just need to
   encrypt the message by using public key and return the encrypted messages to
   user.

4. About the algorithms, in V cycle, Zaqar will introduce the AES-256 encryption
   at first. In next cycles, Zaqar can suppot asymmetric encryption to let user
   upload public key and keep the private key by their own.

.. note::

   About the option of encryption algorithms and keys, Zaqar would support
   specify them throught more metadatas of queues, but it will be done in next
   serveral cycles. In Victoria, we will choose one algorithm (like AES256) to
   support and support to storage the keys by Zaqar itself or other service like
   Barbican.

API Impact
-----------
Create queue list
POST: /v2/queues/queue_name

  RESPONSE CODE: 200
  REQUEST BODY:
  {
    "_enable_encrypt_messages": true
  }

Drawbacks
---------

The ecryption algorithms will impact the performance of storing messages into backends
and getting the messages from the queue.

This depends on which kind of encryption algorithms we choose and support.

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
  victoria M-2

Work Items
----------

#. Modify api and transport code.
#. Add support for encryption algorithm and key management.
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