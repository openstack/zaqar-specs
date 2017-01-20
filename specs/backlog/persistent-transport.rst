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

- Communication with browsers: Websocket will enhance the communication
  between browsers and Zaqar, a key factor in the integration of Zaqar with
  the OpenStack Dashboard (Horizon).

- Notifications: this type of transport will constitute a better protocol
  for notifications, a feature planned for Kilo as well.

https://blueprints.launchpad.net/zaqar/+spec/persistent-transport

Problem description
===================

Currently there is no way to establish persistent connections in Zaqar.
This spec proposes adding this feature through a websocket implementation.

During Kilo release we refactored the code to make the addition of this driver
feasible, and we worked on adding the queues endpoint.

To provide full support for the data plane, we should add the messages endpoint
and the claims endpoint.

The control plane operations do not require the performance that a websocket
driver can provide, hence we are not going to add websocket support for the
control plane.

Proposed change
===============

The proposed change, as stated in previous sections, is to add a persistent
transport alternative to Zaqar. This can be done by using websocket.

Websocket provide a full-duplex communication channel operating over a single
socket that will remove overhead and significantly reduce complexity.

Fallback
--------

In case WebSocket is not available, the client will fallback to short-polling
the REST API.

Some of the reasons of WebSockets not being available are:

- Firewalls could be configured to kill HTTP connections after a certain
  amount of time.

- Concerns from a security standpoint will lead to killing any traffic
  over port 80 or 443 that doesn't look like HTTP.

Messages serialization
----------------------

A serialization technology is needed in order to transport data. There are
several alternatives, including JSON, MsgPack, Protobuf, Apache Avro and
Cap'n Proto.

In the previous cycle we noted that the whereas MsgPack was the best candidate
-considering overall performance, easiness to use, languages support,
data structures support-, it doesn't have good support for Javascript. Using
MsgPack would impose a significative constraint to the Zaqar use cases,
considering that many web applications nowaways are written with Javascript.
For that reason, we decided to stick with JSON and maybe talk about a more
performant serialization technology in a future change.

Alternatives
------------

- Establish persistent connections enabling WSGI keep-alive
- Long polling

Implementation
==============

Assignee(s)
-----------

Primary assignee: vkmc

Milestones
----------

Target Milestone for completion: Liberty-1

Work Items
----------

* Implement messages controller
* Implement claims controller
* Write messages unit tests
* Write claims unit tests
* Would like to have: write functional tests
* Perform benchmarks with the new driver

WebSocket Libraries
-------------------

After discussing several alternatives, including SockJS, SocketIO, Autobahn,
ws4py and Tornado, it has been decided that Autobahn is going to be used to
implement this feature. More details on this decision will be available in
the `WebSockets wiki`_.

.. _WebSockets wiki: https://wiki.openstack.org/wiki/Zaqar/specs/websockets

Dependencies
============

* https://blueprints.launchpad.net/zaqar/+spec/cross-transport-api-spec

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode


