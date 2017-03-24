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
Notification Delivery Policy
============================

https://blueprints.launchpad.net/zaqar/+spec/notification-delivery-policy

Sometimes, because of network/internet connection issue or the load of the
target services/applications, the notification sent from Zaqar to the
subscriber may fail, so it would be nice if we can support a retry and
user can define the retry policy in the options of subscription.

This feature is only for HTTP/HTTPS subscription way, not including the Email
notification.

Problem description
===================

Now Zaqar didn't have notification delivery policy, that means there is no
retries if endpoint has some problem that can't receive the notification.
For users or applications, it's missing important message without notification.

AWS SNS service has supported the policy, that would be a good reference for
Zaqar[1].

Proposed change
===============

For clarification what we want to do, there is some questions to be answered:

1. When do the notification delivery policy work?

Zaqar only attempts a retry after a failed delivery attempt.
Those situations as a failed delivery attempt:

* HTTP status in the range 500-599.
* HTTP status outside the range 200-599.
* A request timeout.
* Any connection error such as connection timeout, endpoint unreachable, etc.

2. How do the notification delivery policy work?

You can use delivery policies to control not only the total number of retries,
but also the time delay between each retry.

There are total four discrete phases that user can use to set policy of retry:

* Immediate Retry Phase: Also called the no delay phase, this phase occurs
  immediately after the initial delivery attempt. The value user set for
  'Retries with no delay' determines the number of retries immediately after
  the initial delivery attempt. There is no delay between retries in this
  phase.

* Pre-Backoff Phase: The pre-backoff phase follows the immediate retry phase.
  Use this phase to create a set of retries that occur before a backoff
  function applies to the retries. Use the 'Minimum delay retries' setting to
  specify the number of retries in the Pre-Backoff Phase. User can control the
  time delay between retries in this phase by using the 'Minimum delay'
  setting.

* Backoff Phase: This phase is called the backoff phase because user can
  control the delay between retries in this phase using the retry backoff
  function. Set the 'Minimum delay' and the 'Maximum delay', and then select a
  Retry backoff function to define how quickly the delay increases from the
  minimum delay to the maximum delay.

* Post-Backoff Phase: The post-backoff phase follows the backoff phase.
  Use the 'Maximum delay retries' setting to specify the number of retries in
  the post-backoff phase. You can control the time delay between retries in
  this phase by using the 'Maximum delay' setting.

User can create and apply the retry policies to queues and subscriptions when
creating them. So when Zaqar send the notification to those subscribers,
it will use those policies to make retried after initial delivery attempt
failed.

..note::

   By default, if there are retry policies on both queue and subscription,
   Zaqar will use the subscription's policy. If user don't want the override
   he can set the ignore_subscription_override=True in _retry_policy metadata
   of the queue.

3. What changes will bring into Zaqar?

* First, user can create retry policies in metadata which's in request body of
  queue creation.

* Second, user also can create retry policies in options which's in request
  body of subscription creation. If retry policy are set to both queue and
  subscription, Zaqar will use subscription policy instead of queue policy
  by default, but if ignore_subscription_override is True in queue's metadata
  then Zaqar will still use the retry policy in queue.

* Third, change the logic of notification, when fail is existing when sending
  notification to endpoint, then use the policy which has binded to this
  subscription to make retries, if subscription doesn't have retry policy, then
  Zaqar will use the policy in queue's metadata.

Since 'metadata' in queue and 'options' in subscription are dict, the data
model of retry policy should be like this::

  {
     '_retry_policy': {
                         'retries_with_no_delay': <Integer value, optional>,
                         'minimum_delay_retries': <Integer value, optional>,
                         'minimum_delay': <Interger value, optional>,
                         'maximum_delay': <Interger value, optional>,
                         'maximum_delay_retries': <Integer value, optional>,
                         'retry_backoff_function': <String value, optional>,
                         'ignore_subscription_override': <Bool value, optional>
                      }
  }

  'minimum_delay' and 'maximum_delay' mean delay time in seconds.
  'retry_backoff_function' mean name of retry backoff function.
  There will be a enum in Zaqar that contain all valid values. At first step,
  Zaqar only supports one function: 'linear'.
  If value of retry_policy is empty dict, that Zaqar will use default
  value to those keys:
  * retries_with_no_delay=3
  * minimum_delay_retries=3
  * minimum_delay=5
  * maximum_delay=60
  * maximum_delay_retries=3
  * retry_backoff_function=linear
  * ignore_subscription_override=False

..note::

   Retry policy is only applied for webhook(http/https) subscriber.
   Zaqar will support linear retry backoff function only in Pike release,
   it will increase the delay time linearly within 10 times from minimum delay
   to maximum delay. We will support more retry backoff functions in future.
   For avoiding the users receive notification message repeatedly
   if connection is reconnected during retry process, once Zaqar sends the
   notification successfully when doing retry, it will stop the whole retry
   process.

Example
-------

For better to know how does this work, there is an example case.
Assume user use the default value of retry policy in a queue A,
and create a subscription with HTTP subscriber: http://192.168.1.100:8080.
When Zaqar send the notifications to this subscriber, it get a response with
HTTP code 500, then the retry policy will go to work:

Phase 1: Immediate Retry. Zaqar will call the subscriber with no delay by 3
         times. If there is no one successful, then go to Phase 2. If one of
         retries is successful, the retry process will be end.

Phase 2: Pre-Backoff. According to the minimum_delay_retries and minimum_delay.
         Zaqar will make a total of 3 retries with a 5 second delay between
         each retry. If there is no one successful, then go to Phase 3.
         If one of retries is successful, the retry process will be end.

Phase 3: Backoff. By using the linear function, Zaqar will make the delay
         between retries to increase at a constant rate over the course of the
         backoff phase. So as the constant rate is 5, Zaqar will call
         subscriber 12 times with delay time:
         [5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60].
         If there is on one successful, then go to Phase 4.
         If one of retries is successful, the retry process will be end.

Phase 4: Post-Backoff. According to the maximum_delay and
         maximum_delay_retries, Zaqar will make a total of 3 retries with a
         60 second delay between each retry. After this phase, no matter the
         call is successful or not, the retry process will be end.

Drawbacks
---------

N/A

Alternatives
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao (sxmatch1986@gmail.com)

Milestones
----------

P-3

Work Items
----------

* Add verification for retry policy when creating queue and subscription.
* Change the notification process for applying the policy.
* UTs for this feature.


Dependencies
============

[1]: http://docs.aws.amazon.com/sns/latest/dg/DeliveryPolicies.html
