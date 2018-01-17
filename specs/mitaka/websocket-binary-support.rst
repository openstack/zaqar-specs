..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===================================================
 Support for binary data in the websocket transport
===================================================

https://blueprints.launchpad.net/zaqar/+spec/websocket-binary-support

Implement support for binary communications over websocket transport which will
allow to reduce network traffic between Zaqar and it's clients and also to
increase performance.

Problem description
===================

Currently websocket transport uses only text messages packed in JSON format to
exchange data with clients.

An attempt to send binary data to Zaqar's websocket transport will lead to
receiving 'bad request' (HTTP error 400) response from Zaqar.

But websocket protocol actually allows communication via binary messages.
Data packed(serialized) in binary format can be smaller and also may take less
CPU time to pack/unpack it.

By using binary messages for communications it's possible to reduce network
traffic between Zaqar and it's clients and also to increase Zaqar's
performance. It can also increase client performance if the client is not
written in JavaScript.

Proposed change
===============

We can serialize/deserialize data via MsgPack technology which was already
proposed and approved in persistent-transport_ specification.

.. _persistent-transport: https://github.com/openstack/zaqar-specs/blob/master/specs/kilo/approved/persistent-transport.rst

Deserialization of incoming messages to Zaqar
---------------------------------------------

As you can see in `protocol.py#L55`_, Zaqar already can distinguish binary and
text messages.

.. _protocol.py#L55: https://github.com/openstack/zaqar/blob/stable/liberty/zaqar/transport/websocket/protocol.py#L55

So we will make Zaqar try to deserialize each received binary message via
'msgpack' library into a python object, just like Zaqar currently deserializes
JSON messages via 'json' library. After successful deserialization the
message can be processed as usual.

Serialization of response and subscription messages from Zaqar
--------------------------------------------------------------

The response from Zaqar will be serialized in the same format as the incoming
request.

The subscription messages from Zaqar will be serialized in binary or text
format based on the format of subscription create request.

Let's take a look at the case where the client suddenly changes it's messaging
format while connection is open and subscription is still alive. There might
be a problem when the client will not be able to deserialize the messages
coming from the old subscription, because it has switched the format by
changing some boolean variable (switch). But this situation is nearly
impossible and, if not, the problem is solvable:

#. It's hard to imagine a situation when it might be useful for the client to
   change it's serialization format. Such client will probably never exist.
#. Even if such client will exist in the future, the client developer will
   take care of it by adding the code checking the format of each incoming
   message and deserializing the message accordingly inside "on message"
   method. Zaqar does exactly this, for example. All websocket implementations
   provide a convenient way to determine the format of each particular message.
#. Even if the client developer will not take care of this potential problem
   (he is inexperienced, for example) and will make the client just expect
   all incoming messages in the new format after switching while the old
   subscription is still alive, the client will crash on the **first incoming
   message from this subscription** and the problem will not go unnoticed.


Websocket html client example upgrade
-------------------------------------

The websocket html `client example`_ will be changed to make it also being able
to communicate with Zaqar via binary messages.

.. _client example: https://github.com/openstack/zaqar/blob/stable/liberty/examples/websocket.html

Drawbacks
---------

The performance of web clients on JavaScript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Seems like MsgPack serialization/deserialization works slower than JSON in
JavaScript. Well, no surprise since JSON abbreviation expands to "JavaScript
Object Notation".
It's a drawback for Zaqar web clients as it decreases their performance.

How the change addresses this drawback:

#. The server is one and clients are many. The server processes much more
   messages than each particular client. So there are many cases when it's
   reasonable to use binary serialization to increase server performance by
   sacrificing performance of JavaScript clients.

#. The additional delays on JavaScript clients are slightly compensated by
   faster responses from the server.

#. JavaScript clients can always use good old text format for communications
   with the server, while other clients can use binary format with the same
   server simultaneously.

Alternatives
------------

These alternatives completely exclude the nearly impossible case where the
client may suddenly switch it's messaging format, while the connection is open
and the old subscription is still alive, and while proper deserialization of
incoming messages to the client is managed by some client's boolean variable
(switch).

Alternatives of serialization of response and subscription messages from Zaqar
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Optional parameter approach
"""""""""""""""""""""""""""

The response from Zaqar will be serialized in the same format as the incoming
request.

The subscription messages from Zaqar will be serialized in binary
or text format based on the new optional parameter passed in the message
body of subscription create request. We can add this parameter to websocket
API, name it ``accept``, allow it to have only two possible values: ``json``
and ``msgpack`` and make ``json`` value as default if ``accept`` parameter was
not passed.
By making ``accept`` parameter optional, we are achieving backward
compatibility.

Subscription request message example before implementation::

    {
        'action': 'subscription_create',
        'headers': {'Client-ID': 31209ff3-ba03-4cec-b4ca-655f4899f8aa,
                    'X-Project-ID': superproject},
        'body': {'queue_name': 'superqueue', 'ttl': 3600}
    }

Subscription request message after implementation can be the same as before or
it can also pass ``accept`` parameter explicitly::

    {
        'action': 'subscription_create',
        'headers': {'Client-ID': 31209ff3-ba03-4cec-b4ca-655f4899f8aa,
                    'X-Project-ID': superproject},
        'body': {'queue_name': 'superqueue', 'ttl': 3600, 'accept': 'json'}
    }

First message approach
""""""""""""""""""""""

The response and subscription messages from Zaqar will be serialized in binary
or text format based on in which format the particular client has sent it's
first successfully parsed message to Zaqar since connecting.

Once the format of communications between Zaqar and the client was
established by that way, the client will not be able to change it, unless the
client will reconnect to Zaqar and send the first message in other format.

Zaqar configuration approach
""""""""""""""""""""""""""""

The response and subscription messages from Zaqar will be serialized in binary
or text format based on the boolean variable in Zaqar server configuration.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ubershy

Milestones
----------

Target Milestone for completion:
  Mitaka-2

Work Items
----------

None

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

