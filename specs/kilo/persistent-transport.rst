..
  This template should be in ReSTructured text. The filename in the
  git repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be
  named awesome-thing.rst.

  Please do not delete any of the sections in this template.  If you
  have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html To test
  out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

====================
Persistent transport
====================

Persistent connections allow a single TCP connection to send and receive
multiple requests/responses instead of opening a new TCP connection for
each pair. When an application opens fewer TCP connections and keeps them
open for a longer period, it causes less network traffic, use less time
establishing new connections, and allows the TCP protocol to work more
efficiently.

Currently Zaqar has support for a non-persistent transport, leaving aside
some use cases that would be covered in a more efficient and reliable way
with a persistent transport solution.

Some use cases that will be positively impacted by this change are:

- Communication with browsers: Websockets will enhance the communication
  between browsers and Zaqar, a key factor in the integration of Zaqar with
  the OpenStack Dashboard (Horizon).

- Notifications: this type of transport will constitute a better protocol
  for notifications, a feature planned for Kilo as well.

https://blueprints.launchpad.net/zaqar/+spec/persistent-transport

Problem description
===================

Currently there is no way to establish persistent connections in Zaqar.
This spec proposes adding this feature through a websockets implementation.

Proposed change
===============

The proposed change, as stated in previous sections, is to add a persistent
transport alternative to Zaqar. This can be done by using websockets.

Websockets provide a full-duplex communication channel operating over a single
socket that will remove overhead and significantly reduce complexity.

Currently the tranport layer in Zaqar is tightly bound to the WSGI transport,
making this addition a complex task.

Fallback
--------

In case WebSockets are not available, the client will fallback to short-polling
the REST API.

Some of the reasons of WebSockets not being available are:

- Firewalls could be configured to kill HTTP connections after a certain
  amount of time.

- Concerns from a security standpoint will lead to killing any traffic
  over port 80 or 443 that doesn't look like HTTP.

Messages serialization
----------------------

A serialization technology is needed in order to transport data. There are
several alternatives, including MsgPack, Protobuf, Apache Avro and Cap'n Proto.

Given that a high priority is being given to browsers compatibility, MsgPack
is the best choice between those. Remaining alternatives doesn't have
out-of-the-box Javascript support, making it harder to ensure compatibility
with most web applications.

Alternatives
------------

- Establish persistent connections enabling WSGI keep-alive
- Long polling

Implementation
==============

Assignee(s)
-----------

Primary assignee: vkmc

Secondary assignee: cpallares

Milestones
----------

Target Milestone for completion: < K-1

Work Items
----------

* Implement the common API layer
* Define the wire protocol for persistent transports
* Implement the driver
* Implement queues and messages controller
* Implement claims controller
* Implement pools, flavors and stats controllers

WebSocket Libraries
-------------------

After discussing several alternatives, including SockJS, SocketIO, Autobahn,
ws4py and Tornado, it has been decided that ws4py is going to be used to
implement this feature. More details on this decision will be available in
the `WebSockets wiki`_.

.. _WebSockets wiki: https://wiki.openstack.org/wiki/Zaqar/specs/websockets

Dependencies
============

* https://review.openstack.org/#/c/122425/
* https://blueprints.launchpad.net/zaqar/+spec/cross-transport-api-spec

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode


