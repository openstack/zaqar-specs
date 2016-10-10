..
  This template should be in ReSTructured text. The filename in the
  git repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be
  named awesome-thing.rst.

  Please do not delete any of the sections in this template.  If you
  have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html To test
  out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==========
OSProfiler
==========

OpenStack consists of multiple projects, composed of multiple services,
working together to make different services available for the users.
In case something is not working as expected, its a really complex task
to determine what is going wrong and to locate the module causing the
bottleneck.

The OSProfiler tool aims to solve this issue by making possible to generate
one trace per request affecting all involved services and build a tree of
calls.

More details on how OSProfiler works can be find in the
`OSProfiler Readme <https://github.com/openstack/osprofiler/blob/master/README.rst>`_

This is even more important for a project like Zaqar in which performance
in communications is a key feature.

https://blueprints.launchpad.net/zaqar/+spec/osprofiler

Problem description
===================

Currently there is no way to easily detect Zaqar's bottlenecks.

Now that some projects have successfully integrated with OSProfiler,
it is important to enable OSProfiler for Zaqar as well and contribute
by providing a continuous trace information view for a request traversing
the different services.

This spec proposes adding the OSProfiler tool to Zaqar.

Proposed change
===============

The proposed change is to integrate the OSProfiler tool to Zaqar.
According to the requirements for this addition detailed in
`OSProfiler Readme <https://github.com/openstack/osprofiler/blob/master/README.rst>`_,
this shouldn't be a complex task. The steps required to do this are detailed
in the `Work Items`_ section.

Main functional change for Zaqar server:
https://review.openstack.org/#/c/141356/

Alternatives
------------

- Not to add the OSProfiler tool

Implementation
==============

Assignee(s)
-----------

Primary assignee: zhiyan

Other assignee: wangxiyuan(wxy)

Milestones
----------

Target Milestone for completion: O-2

Work Items
----------

* Add initialization of OSProfiler when Zaqar server starts

* Add tracing support for WSGI

* Add tracing support for MongoDB

* Add tracing support for Redis

* Add tracing support for SQLAlchemy

* Add tracing support for Pooling

* Add profiler CONF group

  * Enable or disable profiling fully

  * Enable or disable MongoDB tracing

  * Enable or disable Redis tracing

  * Enable or disable SQLAlchemy tracing

  * Enable or disable Pooling tracing

  * Configure HMAC key(s) to protect trace data

* Add profile support for client

* Test the "resting" overhead when profiling is disabled and without feature

* Update document(s) to make sure new options are well documented.

  .. note::

    Usually DB requests create a lot of trace info, so its important to
    be able to enable/disable those on demand

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode
