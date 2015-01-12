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
 Zaqar's notification semantics
================================

One of Zaqar's goals, from an API semantics perspective, is to provide
support for push notifications - or subscriptions, if you will. After
having added support for all the queuing related semantcis to Zaqar's
API, we can start adding support for other things that sit on top of
the existing semantics like the ones mentioned in this spec.

https://blueprints.launchpad.net/zaqar/+spec/notifications

Other references:
- https://etherpad.openstack.org/p/marconi-notifications-brainstorm
- https://etherpad.openstack.org/p/juno-marconi-notifications-on-marconi
- https://etherpad.openstack.org/p/IcehouseMarconiNotificationService
- https://www.dropbox.com/s/d2q1viz4ejebg24/marconi-notifications.png


Problem description
===================

Zaqar's support for queuing semantics that allow users to publish and
consume messages to and from the service. These semantics cover
several use-cases but they leave out others like 'listening' for
messages. There are, as well as for prod/cons scenarios, many
use-cases for pub/sub that need to be supported. Some examples of such
use-cases are: mobile push notifications, support for events
triggering, meetering, etc.

In order to cover the above scenarios, Zaqar needs to provide an API
that will allow users for subscribing to specific queues and get
messages from there without requiring to poll. This will allow for a
better architecture on the client side, a more reliable interaction
between Zaqar and the client in terms of connection management and it
also opens the doors for supporting other scenarios.

Proposed change
===============

The proposal is to extend Zaqar's API with support for subscriptions
without adding a separate service - please see the section below with
regards to both services being merged. This proposal introduces the
high-level changes required for this feature but the detailed API
specification has yet to be written.

Here's a, non-exahustive, list of resources required to support
notifications:

**Subscriptions**: Subscriptions define what the subscriber wants to
listen on. It contains the data of the queues/topics, routing
parameters, maximum retries and other configirations required to
correctly filter messages and keep the high-level guarantees.

A subscription is used by subscribers to listen on things. A non
persistent subscriber - webhooks, sms, mobile push protocols - are
configured in the subscription itself, whereas other subscribers using
persistent connection can load a specific subscription they want to
listen on.

A subscription consists in a object containing the information of the
source, destination, expiration and a custom field that contains
information useful for the notifier. ::

    POST /v2/queues/{queue_name}/subscriptions

    {
        'subscriber': 'http://trigger.me',
        'ttl': 3600,
        'options': {}
    }

* **source**: The source queue for notifications
* **subscriber**: The subscriber URI. This can be any of the supported
  subscriber URI and the implementation must be based on stevedore to
  allow external contributions.
* **ttl**: time to live for this subscription. It's a positive number
  that expresses, in seconds, how long a subscription should exist. If
  omitted the subscription won't expire.
* **options**: publisher specific options.

**Publishers**: Publishers are users and/or third-party resources
publishing messages to the notification endpoints. Users publishing
messages to Zaqar don't need any specialized endpoint for
this. Messages must go through the normal and already implemented
message publishing API. However, third-party publishers require a
specific configuration. Third-party publishers consume messages from
other services and publish them in zaqar's queues.

**Workers**: Publishing tasks need to be executed in a distributed -
or local to the node - fashion, they need to be reliable,
fault-tolerant and atomic. To do so, I've evaluated taskflow as a
valid library for this job. Taskflow has support for
local/distributed, blocking/concurrent task execution. It provides
support for several execution models, persistence layers - including
sqlite, mysql and zookeeper - and it has support for retries. The
alternative to taskflow is writing our on implementation for this
work, which doesn't seem to be worth it at this stage.

Push Protocols
--------------

There are several types of push technologies that could be supported,
therefore the implementation must be generic enough to allow for new
implementations to be done easily and also as external plugins.

Nonetheless, this spec proposes starting with just the first of the
plugins proposed below:

- Webhooks
- Email
- Mobile push
- SMS

Why not 2 services
------------------

Zaqar has stated since its inception that it provides a single common
API for things that other cloud services like AWS have in 2 separate
services. Nonetheless, I think the real motivations for these 2 API's
to be under the same service are:

- Maintenance: It's easier to maintain and scale just 1 service type
  than a group of different services.
- Consistency: It'll help reducing the code needed to implement the
  notification API since they both will be running under the same
  structure and users will know where to find the services easily.
- Performance: Implementing the notification API in a separte service
  will require a communication between that service and the queuing
  service. At least, it'll require a constant polling from the backend
  (if it doesn't support push itself). By having everything under the
  same service, we can trigger notification for messages as they come
  in.

Future Improvements
-------------------

* Support for subscription's confirmation

Alternatives
------------

- Split the above into 2 different services. As mentioned in the
  previous section, 2 services don't seem to be the ideal solution for
  this.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flwang

Secondary assignees:
  flaper87

Milestones
----------

Target Milestone for completion:
  Kilo-2

Work Items
----------

* Work on the detailed API spec
* Write storage code for notifications
* Implement the API on top of the storage code.
* Work on the publishers

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

