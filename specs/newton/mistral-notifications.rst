..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=======================
 Mistral Notifications
=======================

https://blueprints.launchpad.net/zaqar/+spec/mistral-notifications

Allow a message to a Zaqar queue to trigger a Mistral workflow via the Zaqar
notification mechanism.

Problem description
===================

Developers of cloud applications expect to be able to build autonomous
applications in the cloud. That is to say, applications that manage themselves
by accessing the APIs of the cloud to manipulate their own infrastructure.
(Examples of this include autoscaling and autorecovery.) This is one of the
primary differences between a cloud platform and a simple virtualisation
platform (the other being multi-tenancy). There are two parts to this that
require integration, which is the purpose of this blueprint.

The first is that the application must be able to receive information from the
cloud. An example of this would be an Aodh alarm indicating that a server is
overutilised. These notifications must be asynchronous, since the cloud is
multitenant and cannot block waiting for any one user application to
acknowledge it. They must also exhibit queueing semantics with at-least-once
delivery and high durability, since the application may become unreliable if it
misses notifications from the cloud. Not coincidentally, Zaqar offers exactly
these semantics in a public, Keystone-authenticated API that is accessible to
applications, and is therefore a natural choice. For this reason, a number of
OpenStack projects have already started dispatching user notifications to Zaqar
and more are expected in the near future. Already Aodh alarms support Zaqar as
a target, and Heat can push stack and resources events as well as notifications
about user hooks being triggered to Zaqar.

The second is that the application must be able to perform arbitrary, and
arbitrarily-complex actions. This is because in practice the Right Thing to do
in cases like autoscaling and autorecovery is application-specific. There is
also an entire universe of application-specific actions that a user might want
to create. Of course an application can run these actions on a server
provisioned with Nova, but this generally makes things more complex (and
usually more expensive) than they need to be. For example, it is very hard to
host autorecovery code on the servers that are being autorecovered themselves
and still be reliable. Finally, OpenStack makes it difficult to provide
appropriate Keystone credentials to servers provisioned with Nova. Mistral_
solves these problems by providing a lightweight, multi-tenant way of reliably
running potentially long-running processes, with access to the OpenStack APIs
as well as a number of other actions (some of which, like sending email and
webhooks, are similar to Zaqar's notifications).

The missing link to build fully autonomous applications is for messages
(potentially, but not neccessarily originating from the OpenStack cloud itself)
on Zaqar queues to be able to trigger Mistral workflows (potentially, but not
neccessarily calling other OpenStack APIs). This would give developers of cloud
applications an extremely flexible way of plugging together event-driven,
application-specific, autonomous actions.

.. _Mistral: https://wiki.openstack.org/wiki/Mistral

Proposed change
===============

Create a Zaqar notification sink plugin for Mistral. The effect of a
notification to this sink would be to create a Mistral workflow Execution_
(i.e. to trigger a pre-existing Mistral workflow).

The ``subscriber`` URI should be the URL of the Mistral executions endpoint,
with the URI scheme ``trust+http`` or ``trust+https``. For example,
``trust+https://mistral.example.net/v2/executions``. This scheme indicates that
Zaqar should create a Keystone trust that allows it to act on behalf of the
user in making API calls to Mistral in the future. The trust ID will be
inserted into the URL before it is stored in the form
``trust+http://trust_id@host/path``. This form is modelled after `the one used
by Aodh`_.

The trust lifetime should be slightly longer than the TTL of the subscription,
or unlimited if there is no TTL for the subscription. Zaqar must delete the
trust when deleting the subscription.

When sending a notification, Zaqar will retrieve a trust token from Keystone
using its own service user token and the trust ID stored in the URL. The trust
token thus obtained should contain the correct tenant information to then make
the request on behalf of the original user.

Since in future Zaqar may want to make ``trust+http`` requests to other API
endpoints, it should distinguish on more than just the URI scheme. When the
subscription is created, Zaqar should need compare the URI with the Mistral
executions endpoint URL obtained with the help of the Keystone catalog in order
to distinguish between Mistral workflow triggers and ordinary webhooks.
Fortunately, the URL is fixed for a given cloud, so the catalog would probably
only need to be read once and it would be a straight string comparison from
there.

The ``options`` dict should contain the following keys:

* ``workflow_id`` - The ID of the workflow to trigger
* ``params`` - a dict of parameters that varies depending on the workflow type.
  e.g. a "reverse workflow" takes a ``task_name`` parameter to define the
  target task.
* ``input`` - an arbitrary dict of keys and values to be passed as
  input to every workflow execution triggered by this notification.

When creating the Mistral execution, the contents of the message and (later)
the message ID will be passed in the environment (the ``env`` key in the
``params``). This allows the workflow to access the message data, but does not
require it to declare a particular input for it (so the notification can be
used to trigger *any* workflow). The message contents, interpreted as JSON,
will be passed in a Mistral environment variable named ``notification``. When
Zaqar supports passing the message id in a notification, it will be sent as the
Mistral environment variable ``notification_id``. If these names conflict with
the ``env`` passed by the user in ``params``, the user-provided data will be
overwritten with that received in the message.  Any other keys in the user's
``env`` will be preserved. If the user does not specify an ``env``, one will be
created.  The ``input`` dict, ``workflow_id`` and all other ``params`` will be
passed through unmodified.

While all the data is available to do a raw HTTP request, it is preferable if
these calls are made through the python-mistralclient library.

.. _Execution: http://docs.openstack.org/developer/mistral/developer/webapi/v2.html#executions
.. _the one used by Aodh: http://docs.openstack.org/developer/aodh/architecture.html#trust-http

Alternatives
------------

Instead of a push model, where Zaqar takes messages and notifies Mistral, it
would also be possible to use a pull model where Mistral polls Zaqar topics for
messages. However, while the Zaqar notification implementation already exists,
there is no such existing component in Mistral that would be suitable for
polling for triggers. It would need to poll large numbers of topics in
different tenants. A similar design was considered and rejected for the
notification feature of Zaqar; the same arguments apply here.

An alternative authentication method might be to use pre-signed URLs, which are
on the `Mistral roadmap`_. This might be quicker to implement, but in the
longer term, Keystone trusts are probably preferable.

Instead of whitelisting the Mistral executions URL, the ``trust+http`` scheme
could be used to make requests to any OpenStack endpoint. However, in general
the correct method of combining static information from the ``options`` dict
with the contents of the message to obtain the call parameters will be
different for every API. Since Mistral can already call most OpenStack APIs and
supports a language (YAQL) for calculating the arguments using data from the
notification and other input, the simplest way to achieve this is for the user
to encapsulate any other OpenStack API call they wish to make in a Mistral
workflow (which also allows them to define custom error handling).

It would be nice if there were a way to identify an OpenStack resource with a
URI without necessarily requiring a URL (containing redundant information about
the location of the endpoint). AWS uses an `unofficial URN-like identifier`_
with an arn: (instead of urn:) scheme for this purpose. Something similar might
be useful in other contexts in OpenStack too (for example, in Heat we would
like to be able to distinguish between files in Swift containers or Glare links
and ordinary HTTP URLs for the purposes of uploading user data, although there
is some precedent for using ``swift+http`` as the scheme in the Swift case).
However, this would require, at a minimum, wide cross-project agreement (and
arguably IANA registration). There are no existing examples of anything like
this in OpenStack.

.. _Mistral roadmap: https://wiki.openstack.org/wiki/Mistral/Roadmap
.. _unofficial URN-like identifier: http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html

Implementation
==============

Assignee(s)
-----------

This is one of those blueprints where I'm throwing it out there to see who
picks it up.

Milestones
----------

Target Milestone for completion:
  Newton-3

Work Items
----------

* Implement the Mistral notification plugin
* Create a keystone trust and store its ID in the URI when setting up a
  ``trust+http(s)`` notification. Delete the trust again when the notification
  is deleted.
* Add the ability to distinguish between Mistral URLs and other
  ``trust+http(s)`` URLs in the notification URI

Dependencies
============

We won't be able to pass the message ID until
https://review.openstack.org/#/c/276968/ or something equivalent merges.
However, since it can be added to the Mistral environment later without
rewriting any existing workflows (to declare a new input), this is in no way a
blocker.

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

