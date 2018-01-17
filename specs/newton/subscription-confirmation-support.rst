..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===================================
 Subscription confirmation support
===================================

https://blueprints.launchpad.net/zaqar/+spec/subscription-confirmation-support

Support subscribers confirm that whether they want the notifications or not
before Zaqar sends them.

Problem description
===================

Currently Zaqar can post notifications to the subscriber once the subscription
is created. It doesn't need the subscriber to confirm the subscription.
This will lead a problem that users could send junk information that whether
subscriber wants or not.

Proposed change
===============

With `Amazon SNS`_ as reference, we will support to send HTTP POST request
that contains subscription confirmation information(e.g. message type,
pre-signed confirmation URL, signature information) to subscription endpoint.
The endpoint needs to use the URL in notification to confirm subscription.
When the subscription is deleted, Zaqar server will send a notification that
the subscription with ID has been deleted. This mechanism tells endpoint no
need to wait notifications any more.

* The message types will be SubscriptionConfirmation, Notification
  and UnsubscribeConfirmation.
* For backward compatibility, we also should add new option in Zaqar
  configuration file like this::

    [notification]
    require_confirmation=true/false

If the user omits this option in config, it will be 'false' by default.
In further, we may change the default value to 'true' to let subscription
more secure.

So the workflow will be this in webhook:

#. User creates subscription in Zaqar.
#. Zaqar sends a HTTP POST request with subscription confirmation information.
   This request is like this:

   POST URL::

     http://endpointUrl

   Request Body::

     {
         'MessageType': 'SubscriptionConfirmation',
         'Message': 'You have chosen to subscribe to the queue: xxx.',
         'URL-Signature': 'xxxx',
         'URL-METHODS': 'PUT',
         'URL-PATHS': '/v2/queues/{queue_name}/subscriptions/{subscriptions_id}/confirm',
         'X-PROJECT-ID': 'xxxx',
         'URL-EXPIRES': '3600-01-01T00:00:00',
         'SubscribeURL': 'https://zaqar_server/v2/queues/{queue_name}/subscriptions/{subscriptions_id}/confirm',
         'SubscribeBody': {'confirmed': True},
         'UnsubscribeBody': {'confirmed': False},
    }
#. Subscriber confirms this subscription by using those information
   in request body, and sends the pre-signed confirmation request to Zaqar.
   It's a new endpoint in Zaqar.
   This request is like this:

   PUT URL::

     https://zaqar_server/v2/queues/{queue_name}/subscriptions/{subscriptions_id}/confirm

   Request Body::

     {
         'confirmed': True
     }

   Meanwhile, HTTP headers are also required by using pre-signed feature
   in Zaqar. If subscriber wants to unsubscribe this subscription, just needs
   to set 'confirmed' to False.
   If Zaqar processes this request successfully, Zaqar will response with:

   Response code::

     204

   If Zaqar has failed processsing this request because of some error, it will
   return error message in response body.
#. Zaqar begins to send notifications to this endpoint after confirmation
   workflow is finished.

.. note::

   If Zaqar server didn't receive the response or subscriber didn't sent it
   since something wrong, subscription will keep 'confirmed' as false, and
   Zaqar will support to resend confirmation request by calling subscription
   creation API again.

So the workflow will be this in email:

#. User creates subscription in Zaqar.
#. Zaqar sends an email with subscription confirmation page link.
   Email context is like this::
   "You have chosen to subscribe to the queue: {queue name}."
   "This queue belongs to project: {project id}."
   "To confirm this subscription, click or visit this linke below:"
   "{confirmation link}"
#. When subscriber visits this link, he gets a page which calls
   Zaqar by ajax request to confirm the subscription as same as
   webhook way.
#. The page shows information about was the subscription successfully
   confirmed or not.
#. Zaqar begins to send notifications to this endpoint.

.. note::

   About the confirmation page, Zaqar will host a sample page in Zaqar
   tree, but it's not a part of Zaqar service. Cloud administrator should
   provide hosting for this page manually. After this page is hosted
   successfully, cloud administrator should specify this page location
   in Zaqar config "external_confirmation_url".

Drawbacks
---------
None

Alternatives
------------
Another option is when subscriber creates a subscription, he also need to
call Zaqar's confirmation API to confirm it before confirmation is expired.
In this option, Zaqar will not send any confirmation notification by itself.

Data model impact
-----------------
Add new status "confirmed" into subscription model.

Comparison AWS SNS API and Zaqar API
------------------------------------
For developer to better understand this feature, this spec compares SNS API
and Zaqar API what difference between them.

Subscription Confirmation Request
SNS is using GET request like this::

  GET https://sns.us-west-2.amazonaws.com/?Action=ConfirmSubscription&TopicArn={Topic}&Token={Token}

Zaqar is using PUT request like this::

  PUT https://zaqar_server/v2/queues/{queue_name}/subscriptions/{subscriptions_id}/confirm

Request Body::

  {
      'confirmed': True
  }


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wangxiyuan<wangxiyuan@huawei.com>

Secondary assignee:
  wanghao<wanghao749@huawei.com>

Milestones
----------

Target Milestone for completion:
  Newton-3

Work Items
----------

#. Implement subscriber confirmation API in wsgi.
#. Update subscription resource to add new property "confirmed"
   on MongoDB driver.
#. Add new config options.
#. Update notification to send confirmation message in way email.
#. Implement subscriber confirmation API in websocket.
#. Update subscription resource on other drivers.
#. Update notification to send confirmation message in way webhook.
#. Add message type.

Dependencies
============

_`Amazon SNS`: http://docs.aws.amazon.com/sns/latest/dg/SendMessageToHttp.html
