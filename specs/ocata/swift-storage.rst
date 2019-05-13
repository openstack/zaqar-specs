..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=====================
Swift storage backend
=====================

https://blueprints.launchpad.net/zaqar/+spec/swift-storage

Swift is a widely deployed object storage solution for OpenStack. While it's
not particularly designed for high throughput of lots of small objects, which
is what Zaqar mostly does, it scales really well, and is highly fault tolerant.
It would also provide an alternative to people that want to avoid to deploy
and manage MongoDB.

Driver description
==================

The driver only implements the data part, SQL being available for the
control driver. Containers are used per entity and per project, to provide
multitenancy.

Proposed change
===============

The driver uses one global service user, and stores all messages in one
project. It uses suffix on container names to scope them. It will use mostly
one container per entity type, plus other for additional indexes. We may need
to increase or decrease the number of containers used, to find the appropriate
performance balance.

It may be worth marking the driver experimental first, so that we can change
the storage model without caring about migrating data.

What are the driver's guarantees in terms of reliability?
=========================================================

We rely on Swift being configured appropriately to provide message reliability.
By default objects are replicated 3 times, which offers good guarantees.

What are the driver's guarantees in terms of scalability?
=========================================================

Again, we rely on Swift scalability guarantees here.

What are the driver's guarantees in terms of interoperability?
==============================================================

We use publicly available Swift APIs, so the data is easily auditable and
recoverable behind Zaqar.

What are the driver's guarantees in terms of openness?
======================================================

We use python-swiftclient to access Swift, which uses the Apache 2.0 license.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  therve


Milestones
----------

Target Milestone for completion:
  Ocata-3

Work Items
----------

* Implement storage driver, and make unit tests pass.
* Add a voting gate against Swift.
* Possibly make some performance testing to tweak how objects are stored.


Dependencies
============

- Depend on Swift fix for if-none-match: https://review.opendev.org/395582

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

