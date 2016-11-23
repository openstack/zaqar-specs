========================================
 Finish client support for v1.1 features
========================================

https://blueprints.launchpad.net/zaqar/+spec/finish-client-support-
for-v1.1-features

Several endpoints have been added without being reflected on
the client side. This endpoints include:

- Pools
- Flavors
- Notifications

Adding support for these features would make developing applications with Zaqar
easier.

Problem description
===================

Python-zaqarclient currently has some missing features that need to be
added. This spec describes which endpoints are missing and how those are going
to be added to the client side.

Proposed change
===============

Based on the endpoints that are currently missing to the python-zaqarclient
v1.1,the proposed change is to add support for the following features:

1. Pools:

- pool_update
- pool_list

2. flavors:

- flavor_update
- flavor_list

3. Notifications

Alternatives
------------

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  dynarro

Milestones
----------

Target Milestone for completion:
  Liberty-2

Work Items
----------

Add support to:

- pools
- flavors
- notifications

Add unit testing for:

- pools
- flavors
- notifications

Add developers documentation for:

- pools
- flavors
- notifications

Dependencies
============

None

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

