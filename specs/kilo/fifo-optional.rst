..  This template should be in ReSTructured text. The filename in the
  git repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be
  named awesome-thing.rst.

  Please do not delete any of the sections in this template.  If you
  have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html To test
  out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

============================
Make FIFO guarantee optional
============================

FIFO has been one of Zaqar's most wanted guarantees. Besides
reliability, durability and other latency requirements, users have
always requested FIFO to be there. This feature, however, has proven
to be an issue for some scenarios that don't necesarily require it,
hence this proposal.

https://blueprints.launchpad.net/zaqar/+spec/make-fifo-optional

Problem description
===================

FIFO, despite being a great guarantee to have, brings in some scale
and performance issues that Zaqar is not willing to accept as the
default behavior. This spec proposes making FIFO optional and letting
drivers capable of supporting such scenario to do so.

Proposed change
===============

The proposed change, as stated in previous sections, is to make FIFO
optional. It is possible to do so through `flavors`.

Not all store drivers are capable of supporting FIFO but those who
are, will have FIFO listed in their supported capabilities and such
capabilities will be exposed through the `flavors` ones. However, it's
not as straightforward as it seems. See the `Work Items`_ section for
a list of required changes that will make this possible.

Drawbacks
---------

As a side effect of this change, we'll have to relax the delivery
guarantee for pub-sub. The reason being that walking through the queue
won't prevent consumers to skip messages.

Alternatives
------------

- Keep it as is
- Remove FIFO completely

Implementation
==============

Assignee(s)
-----------

Primary assignee: flaper87

Milestones
----------

Target Milestone for completion: K-1

Work Items
----------

* Allow driver to expose what features they support
  * Each driver supporst a set of features and this set needs to be accessible from the upper layers
* Standardize the *supported* capabilities
  * Make a list of supported capabilities
* Add a way to pass capabilities down to the driver
  * We can do this when the driver is initialized since we know what the capabilities are at that time.
* Support both, FIFO and non-FIFO, post methods.


Dependencies
============

* https://review.openstack.org/#/c/126531/

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode
