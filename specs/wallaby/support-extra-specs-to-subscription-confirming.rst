..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==============================================
Support Extra Specs to Subscription-confirming
==============================================

https://blueprints.launchpad.net/zaqar/+spec/support-extra-specs-to-subscription-confirming

This requirement came from a true scenariowhen people use the subscription
function of Zaqar, for example, text messages, they need to return some extra
information like message authentication code in confirming process.
Now Zaqar still cant support it, that will impact the usage of subscription.

Problem description
===================

Currently, Zaqar can't support extra information in subscription
confirming process. This will block user to input information that
is needed when subcription is confirming. There is a true case that
came from Zaqar's user. They want to input the message authentication
code into Zaqar when the subscription is confirming, the code will be
used to identify the subscriber. So Zaqar should support this kind of
mechanism.

Proposed change
===============

1. Introduce a key-value called "extra_spec" in confirming request body.

2. Introduce a driver mechanism to let vendors to implement what they
   want to do with extra_spec information.

API Impact
-----------

Subscription confirming request:

.. code-block::

  PUT: /v2/subscriptions/subscription_id/confirm

    RESPONSE CODE: 204
    REQUEST BODY:
    {
      "confirmed": true,
      "extra_spec": {"message_authentication_code": "xxxxxx"}
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
  wallaby M-2

Work Items
----------

#. Modify api and transport code.
#. Add driver mechanism to handler the extra spec.
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
