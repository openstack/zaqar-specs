..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==============================
More reserved queue attributes
==============================

https://blueprints.launchpad.net/zaqar/+spec/more-reserved-queue-attributes

Currently Zaqar supports setting metadata/attributes when user creating queue.
However, the metadata/attributes are not used very much by Zaqar itself.
Now we only support '_flavor' but it would be nice if we can support
more attributes to make the queue more flexible.

Problem description
===================

Now Zaqar supports max_messages_post_size, default_message_ttl,
default_claim_ttl and default_claim_grace. All of them defined in Zaqar's
configuration file. That means it's global setting, all queues have to share
the same configurations. It's not really flexible from the end user
perspective. So it would be nice if we can let end user set them and use
the default value in configuration file if they are not defined.

Proposed change
===============

The changes are not complicated, which may change two actions: Post messages
and create claim for a queue.

1. Post Message:
   At line stable/liberty/zaqar/transport/wsgi/v2_0/messages.py#L182, we need
   to get the queue's metadata so as to get the queue's max_messages_post_size
   and ttl defined by user when creating the queue. If those aren't exist,
   then just use the global configuration defined in configuration file.

2. Create Claim:
   Similar thing like above, we need to get the queue's metadata at line
   stable/liberty/zaqar/transport/wsgi/v2_0/claims.py#L81 and then use that
   to do the check.

Drawbacks
---------

It may introduce a tiny performance impact for message posting and claim
creating. For example, instead of just checking if the queue is existing or
not, Zaqar need to get the queue's metadata so that to get those attributes.
But actually, we can use 'get_metadata' replace 'exists' to check if the queue
exist or not, and they're calling the same collection.find_one() method, so
technically, there is no performance impact.

Alternatives
------------

Just keep current way, let all the queues share the same configurations.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flwang (flwang@catalyst.net.nz)

Work Items
----------

1. Support max_messages_size and ttl as queue attributes
2. Support claim_ttl and claim_grace as queue attributes


Dependencies
============

N/A

