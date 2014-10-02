..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=============================
 The title of your blueprint
=============================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/zaqar/+spec/example

Introduction paragraph -- why are we doing anything?

Driver description
==================

A detailed description of the driver.

Proposed change
===============

Here is where you cover the change you propose to make in detail. How do you
propose to solve this problem?

If this is one part of a larger effort make it clear where this piece ends. In
other words, what's the scope of this effort?

What are the driver's guarantees in terms of reliability?
=========================================================

"Zaqar aims to be a fully-reliable service, therefore messages should  never be
lost under any circumstances except for when the message's  expiration time
(ttl) is reached..."

What are the driver's guarantees in terms of scalability?
=========================================================

Zaqar aims to be an easily scalable service. Zaqar provides an easy way to
scale web-heads and to balance queue's load across several different storage
clusters. However, the real scalability is provided by the storage itself. If
the storage can't be unlimitedly scaled or it's hard to scale, it'll break this
guarantee. Supported storage must be good for general use-cases and easy to
scale unlimitedly. It's fine to have storage drivers for very specific
use-cases but it's likely not going to be possible to maintain those drivers.

What are the driver's guarantees in terms of interoperability?
==============================================================

In order for applications to easily talk to different clouds supporting Zaqar,
it is necessary for the service to guarantee interoperability as much as
possible. Lots of the interoperability guarantees depend on how the service
itself is deployed. However, the team strives to provide an interoperable
service and guide operators on what the best way to deploy the service is.

What are the driver's guarantees in terms of openness?
======================================================

Zaqar is an open-source software licensed under the Apache 2 license. It aims
to be able to be installed in any deployments and circumstances. In order to do
that, it tries to support technologies that will alow deployers to do so
without any constraint. In order for the team to be able to maintain and
support a storage driver, the driver has to be licensed under an open license
and so has to be the technology the driver supports. The team won't be able to
maintain drivers relying on closed technologies that cannot be tested/deployed.

Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Can list additional ids if they intend on doing substantial implementation work
on this blueprint.

Milestones
----------

Target Milestone for completion:
  Juno-2

Work Items
----------

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Dependencies
============

- Include specific references to specs and/or blueprints in zaqar, or in other
  projects, that this one either depends on or is related to.

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

