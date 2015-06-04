..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

======================
 Pre-Signed URLs
======================

https://blueprints.launchpad.net/zaqar/+spec/pre-signed-url

There's a need to have pre-signed URLs - ala Swift's tempURL - that
would grant temporary access, to non-authenticated users, to specific
queues.

Problem description
===================

In cases where there's a need to allow "outsiders" of the system - i.e
guest-agents - to interact with Zaqar, it's useful to have a way to
grant them access so that they can do whatever is needed without
giving these "outsiders" a username and password or any other kind of
permission in the system.

This access, however, needs to be temporary, revocable and
granular. Outsiders shouldn't get access to more than 1 resource at a
time and the permissions granted should identify what kind of
operations they can execute.

Proposed change
===============

The proposed solution to the aforementioned problem is to use
pre-signed URLs. Services like Swift use a similar approach - called
temp URL there - to provide access to internal resources. The
pre-signed URL consists of a URL that contains a `hash` with the
encoded permissions.

In order to make it easier to read/write this spec, let's split it in
several parts. The first part will describe the parts composing the
URL. The second part will describe how the URL will be generated and
the third part how it'll be consumed.

Pre-Signed URL
--------------

A pre-signed URL ought to contain enough information for it to provide
enough control over the resource being shared, without compromising
security. Authorization is a must have and a huge part of this
URL. Therefore, the information present in this URL has to be
exhaustive in that perspective.

As mentioned in a previous paragraph, the URL contains a hashed piece
of information that serializes the required fields. Those fields are:

* Project: The `keystone` tenant of the entity generating the URL
* TTL: Expiration time (in seconds) for the URL
* Queue: `Zaqar`'s queue name that this URL gives access to
* HTTP Method: The `HTTP` method(s) this `URL` was created for. By
  selecting the HTTP method, it's possible to give either read or
  read/write access to a specific resource.

The above fields will be part of the generated hash but they'll also
be available as public information in the URL. The reason they're
encoded is to have a way to verify that the URL has not been changed
by the user whenever a request is made to `Zaqar`.

The generated hash will be an HMAC-SHA1. To generate such signature,
it is required to have a secret key. In swift, it's possible to have a
key per-account. Unfortunately, that's not possible in Zaqar +
Keystone, therefore this spec proposes adding a new configuration
option that will contain the key to use.::

  [signed_url]
  secret_key = some-very-strong-key


URL Generation
--------------

This spec proposes adding a new endpoint in the queue namespace that
returns the generated signature and expiration time that'll grant
access to this resource. The request and response for this operation:

Request:::

  POST /v2/queues/shared_queue/share HTTP/1.1
  ...

  {
      'methods': ['GET', 'POST']
  }

Response:::


  HTTP/1.0 201 OK
  ...

  {
      'signature': '518b51ea133c4facadae42c328d6b77b',
      'expires': 2015-05-31T19:00:17Z,
      'project': '7d2f63fd4dcc47528e9b1d08f989cc00',
      'url': '/v2/queues/shared_queue/messages',
      'methods': ['GET', 'POST']
  }

This request sets a different expiration date for the URL. Note that
the default method is `GET`.

Request:::

  POST /v2/queues/shared_queue/share HTTP/1.1
  ...

  {
      'expires': 2015-06-19T19:00:00Z
  }

Response:::


  HTTP/1.0 201 OK
  ...

  {
      'signature': '518b51ea133c4facadae42c328d6b77b',
      'expires': 2015-06-19T19:00:00Z,
      'project': '7d2f63fd4dcc47528e9b1d08f989cc00',
      'url': '/v2/queues/shared_queue/messages'
      'methods': ['GET']
  }


This request combines both parameters (methods and expires):

Request:::

  POST /v2/queues/shared_queue/share HTTP/1.1
  ...

  {
      'methods': ['GET', 'POST'],
      'expires': 2015-06-19T19:00:00Z
  }

Response:::


  HTTP/1.0 201 OK
  ...

  {
      'signature': '518b51ea133c4facadae42c328d6b77b',
      'expires': 2015-06-19T19:00:00Z,
      'project': '7d2f63fd4dcc47528e9b1d08f989cc00',
      'url': '/v2/queues/shared_queue/messages'
      'methods': ['GET', 'POST']
  }


Consuming the URL
-----------------

First and foremost, it's important to mention that **NONE** of the
URL headers can/should be modified and/or omitted. As soon as one of
them is, the signature verification will fail and therefore the
request will respond 404.

Requests for pre-signed URLs will be processed by a middleware that
should be placed **before** keystone's middleware. This will allow us
to authenticate the request in advance and skip keystone's
authentication. A request using the signature generated in the
previous section would look like:

Request:::

  GET /v2/queues/shared_queue/messages HTTP/1.1
  Host: zaqar.example.com
  User-Agent: python/2.7 killer-rabbit/1.2
  Date: Wed, 28 Nov 2012 21:14:19 GMT
  Accept: application/json
  Accept-Encoding: gzip
  URL-Signature: 518b51ea133c4facadae42c328d6b77b
  URL-Expires: 2015-05-31T19:00:17Z
  X-Project-Id: 7d2f63fd4dcc47528e9b1d08f989cc00
  Client-ID: 30387f00-39a0-11e2-be4d-a8d15f34bae2

Note that, in the above example, headers were chosen over query
parameters. The main 2 reasons behind this choice are:

1. Consistency with other security related parameters - i.e
X-Project-Id - that are sent in HTTP headers.

2. These new parameters don't belong in the `messages` request and
won't affect `messages` navigation.

Similarly, other requests like the one below can be done.

Request:::

  GET /v2/queues/shared_queue/messages?marker=1355-237242-783&limit=10 HTTP/1.1
  Host: zaqar.example.com
  User-Agent: python/2.7 killer-rabbit/1.2
  Date: Wed, 28 Nov 2012 21:14:19 GMT
  Accept: application/json
  Accept-Encoding: gzip
  URL-Signature: 518b51ea133c4facadae42c328d6b77b
  URL-Expires: 2015-05-31T19:00:17Z
  X-Project-Id: 7d2f63fd4dcc47528e9b1d08f989cc00
  Client-ID: 30387f00-39a0-11e2-be4d-a8d15f34bae2

Filtering and pagination are not part of the signature and fall into
the `read` permissions that were granted on this.

Posting messages will work the same way:

Request:::

  POST /v2/queues/shared_queue/messages HTTP/1.1
  Host: zaqar.example.com
  User-Agent: python/2.7 killer-rabbit/1.2
  Date: Wed, 28 Nov 2012 21:14:19 GMT
  Accept: application/json
  Accept-Encoding: gzip
  URL-Signature: 518b51ea133c4facadae42c328d6b77b
  URL-Expires: 2015-05-31T19:00:17Z
  X-Project-Id: 7d2f63fd4dcc47528e9b1d08f989cc00
  Client-ID: 30387f00-39a0-11e2-be4d-a8d15f34bae2

  ...


Other Notes
-----------

1. In the case of pre-signed URLs, the queue cannot be created
   lazily. This is to prevent cases where queues are deleted and
   users still have a valid URL. This is not a big issues in cases
   where there's just 1 pool. However, if there's a deployment using
   more than 1 type of pool, the lazily created queue may end up in an
   undesired pool and it'd be possible for an attacker to try a DoS on
   that pool. Therefore, whenever a pre-signed URL is created, if a
   pool doesn't exist, one will be created.

2. I'm not a fan of passing the `project-id` around but I can't think
   of another way to do this and still have the ability to preserve
   multi-tenancy without passing the project.

3. I don't like having the key set in the config file. In future
   versions, we could think of making this information part of the
   queue itself. The reason we can't do that right now is because we
   don't have private fields in the metadata. It should be easy enough
   to do it as an enhancement for this feature.

4. As a future enhancement, we could also use Barbican for key management.


Drawbacks
---------

Security issues may be added by this work. We ought to be extra
careful on reviews and create a vulnerability team that is ready to
address any issues that might come up.

Alternatives
------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
    flaper87

Milestones
----------

Target Milestone for completion:
  Liberty-1

Work Items
----------

1. Write utilities to generate the signature with proper tests
2. Add the endpoint that generates the pre-signed URL
3. Create a middleware capable of processing these URL

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

