..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==================================================
 Design wire protocol for non RESTFul transports
==================================================

https://blueprints.launchpad.net/zaqar/+spec/non-rest-wire-protocol

This specs represents the work to be done to design a wire protocol that will
respect the current API v1.1 and that will also be useful for non-RESTFul
transmission protocols.

Problem description
===================

We currently have a well defined protocol that works well for HTTP. As a
project, we're looking forward to welcome more transports that will work for
different use-cases but we don't have a way to support them. This blueprint
aims to define such protocol and the work needed to make it happen.

Proposed change
===============

Based on the existing protocol defined in this wiki page[0] and the rough spec
proposed here[1], I'm proposing we use similar to the one below and complete
the work started by the api-layer blueprint[2].

This blueprint, however, does not propose an API layer that should be used by
*all* transports. Instead, it proposes an API layer that will be used by
transports relying on lower-level transmission protocols like: websocket, raw
tcp, etc.

Ideally, one would expect Zaqar to support existing protocols like
STOMP, AMQP, etc. While this may/may not be possible depending on the
protocol, we should have a protocol that sticks to the service goals
of being simple, straightforward and lightweight. Other protocols
could be implemented as separate transports in the future.

::

    {
        "action": "post_message",
        "header": {
            "User-Agent": "python/2.7 killer-rabbit/1.2",
            "Date": "Wed, 2    8 Nov 2012 21:14:19 GMT",
            "Accept": "application/x-msgpack",
            "Client-ID": 30387f00-39a0-11e2-be4d-a8d15f34bae2,
            "X-Project-ID": 518b51ea133c4facadae42c328d6b77b,
            "X-Auth-Token": "7d2f63fd-4dcc-4752-8e9b-1d08f989cc00"
        },
        "body": {
            ...
        }
    }

[0] https://wiki.openstack.org/wiki/Zaqar/specs/api/v1.1
[1] https://wiki.openstack.org/wiki/Zaqar/specs/zmq/api/v1
[2] https://blueprints.launchpad.net/zaqar/+spec/cross-transport-api-spec

Alternatives
------------

- Don't do anything and let each transport implement it's protocol.
- Implement a transport on top of a different protocol

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flaper87


Milestones
----------

Target Milestone for completion:
  Kilo-1

Work Items
----------

- Define the wire protocol
- Create an API manager that process messages

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

