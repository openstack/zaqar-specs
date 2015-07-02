..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==========================
Email Notification Support
==========================

As a notification service, email is one of the importants subscribers we
should support. Given email is the most way to get messages in the modern
world .

blueprint: https://blueprints.launchpad.net/zaqar/+spec/email-notification

Problem description
===================

Now Zaqar just introduces the notification support in Kilo, but only suppport
webhook driver. The problem is users who do not have a webhook endpoint for
Zaqar to hit, but do have access to email, cannot currently benefit from Zaqar
notifications.


Proposed change
===============

Add a new task driver under /notification/task, which will leverage the built-in
/usr/sbin/sendmail command to send messages.

Proposed REST API request looks like::

    POST /v2/queues/{queue_name}/subscriptions

    {
        'subscriber': 'mailto:fake@gmail.com',
        'ttl': 3600,
        'options': {'subject': 'Alarm', 'cc': {}, 'bcc': {} }
    }


The email notification driver will support the standard mailto protocol, that
said, such as subject, cc and bcc could be a part of the 'subscriber' attribue.
For example:

mailto:someone@example.com?subject=This%20is%20the%20subject&cc=someone_else@example.com&body=This%20is%20the%20body

Meanwhile, we can also support those email fields in the 'options' attribute.

And to avoid spam, we can send a confirmation email firstly with a confirm_token,
then by default the subcription is in 'inactive' status, until the email owner
clicked the URL in the confirmation email. The URL will be like below::

    /v2/queues/{queue_name}/subscriptions?subscriber=mailto:fake@gmail.com&confirm_token=22c4f150358e4ed287fa51e050d7f024


Then Zaqar will update the subscription from 'inactive' to 'active'.


Alternatives
------------

From the design/architecture perspective, if we don't have an email driver for
notification, that means we may need a email-sending-as-a-service to achieve the
same goal. It doesn't make much sense to do that given we can easily do that
by leveraging the mailto in Zaqar's notification service.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

Deployer impact
---------------

Deployers who deploy Zaqar will need to ensure that they have email systems
configured if the email notification is enabled in 'subscriber_types'.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flwang (Feilong Wang)

Milestones
----------

Target Milestone for completion: L-2

Work Items
----------

* Add a task driver for email

Dependencies
============

* Mail transfer agent needs to be configured on the host.

Testing
=======

* Unit tests
* Functional test, like sending to $(whoami)@$(hostname) and then check with 'mail'.
* Manual testing

Documentation Impact
====================

* Feature need to be documented

References
==========

* https://docs.python.org/2/library/email-examples.html

