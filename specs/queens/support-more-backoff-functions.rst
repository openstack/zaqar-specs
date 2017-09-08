..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===============================
Support More Backoff Functions
===============================

https://blueprints.launchpad.net/zaqar/+spec/support-more-backoff-functions

Sometimes, because of network/internet connection issue or the load of the
target services/applications, the notification sent from Zaqar to the
subscriber may fail. That means there is no retries if endpoint has some
problem that can't receive the notification. For users or applications,
it's missing important message without notification. To solve this problem,
Zaqar has supported notification delivery policy in pike release [1].

Problem description
====================

Now that Zaqar has supported only linear retry backoff function in Pike
release,it increases the delay time linearly from minimum delay to maximum
delay. We will support more retry backoff functions in Zaqar now.

Fortunately, AWS SNS service has supported the policy, that would be a good
reference for Zaqar[2].

Proposed change
================

For clarification what we want to do, there are some questions to be answered:

1. How many retry backoff functions Will Zaqar support?

You can choose from four retry backoff functions as follows:

* Linear.
* Arithmetic.
* Geometric.
* Exponential.


2. What difference between these four retry backoff functions?

 As we can see the picture in chapter Backoff Phase in [2], The screen shot
 shows how each retry backoff function affects the delay associated with
 messages during the backoff period. The vertical axis represents the delay
 in seconds associated with each of the 10 retries. The horizontal axis
 represents the retry number. The  minimum delay is 5 seconds, and the
 maximum delay is 260 seconds.


3. The formula of these four retry backoff functions

  Generally, we assume that the minimum delay is MIN, the maximum delay is
  MAX, and the total retry number is NUM. From three element above, we can
  conclude the formula of these four retry backoff functions.

* Linear.

  LINEAR_INTERVAL = ( MAX - MIN ) / NUM
  time1 = MIN
  time2 = MIN + LINEAR_INTERVAL*1
  time3 = MIN + LINEAR_INTERVAL*2
  timen = MIN + LINEAR_INTERVAL*(n-1)
  timeN = MAX

  The Linear retry backoff functions has already been implemented in Pike
  release, we only list here for reference and comparing.

* Geometric

  we assume the common ratio of a geometric series is K:
  time1 = MIN
  time2 = K * MIN
  time3 = K^2 * MIN
  timen = K^(n-1) * MIN
  timeN = K^(NUM-1) * MIN = MAX

  so :

  K^(NUM-1) = MAX/MIN
  lg[ K^(NUM-1) ] = lg( MAX/MIN )
  (NUM-1) * lg(K) = lg( MAX/MIN )
  lg(K) = lg( MAX/MIN ) / (NUM-1)
  K = 10^[  lg( MAX/MIN ) / (NUM-1) ]

  Finally:

  timen = {10^[  lg( MAX/MIN ) / (NUM-1) ]}^(n-1) * MIN

* Exponential.

  we assume the  power exponent of a exponential series is k, coefficient is p:

  the general function of power exponent of a exponential series is:
  y = p * k^x

  time1 = MIN = p*k^1
  timeN = MAX = p*k^NUM
  MAX/MIN = k^(NUM-1)
  lg(MAX/MIN) = (NUM-1)*lg(k)
  lg(k) = lg(MAX/MIN)/ (NUM-1)
  k = 10^[lg(MAX/MIN)/ (NUM-1)]
  p = MIN/{10^[lg(MAX/MIN)/ (NUM-1)]}

  Finally:

  timen = p*k^n
  timen = MIN/{10^[lg(MAX/MIN)/ (NUM-1)]} * {10^[lg(MAX/MIN)/ (NUM-1)]} ^n

* Arithmetic.

  we assume the common difference of a arithmetic series is d:

  time1 = MIN
  time2 = time1 + d*1
  time3 = time2 + d*2
  time4 = time3 + d*3

  timen = timen-1 + d*(n-1)

  timeN = timeN-1 + d*(NUM-1) = MAX

  So:
  timeN - time1 = NUM*(NUM-1)/2*d

  d = 2*(MAX-MIN)/[NUM*(NUM-1)]

  timen - time1 = n*(n-1)/2 * { 2*(MAX-MIN)/[NUM*(NUM-1)] }

  Finally:

  timen  = n*(n-1)/2 * { 2*(MAX-MIN)/[NUM*(NUM-1)] } + MIN

4. What changes will bring into Zaqar?

* First, we will define three functions according to formula of these retry
  backoff functions.

* Second, change the logic of notification, when fail is existing when sending
  notification to endpoint, then use the policy which has binded to this
  subscription to make retries. In Backoff Phase, we will select the retry
  backoff functions according to user choice to resend notification.


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
  gecong (ge.cong@zte.com.cn)

Milestones
----------

Q-2

Work Items
----------

* Add three functions according to formula of these retry
  backoff functions.
* Change the Backoff Phase of notification process for applying the retry
  backoff functions.
* UTs for this feature.


Dependencies
=============


[1]:https://specs.openstack.org/openstack/zaqar-specs/specs/pike/notification-delivery-policy.html
[2]:http://docs.aws.amazon.com/sns/latest/dg/DeliveryPolicies.html
