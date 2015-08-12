..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

============================
Authorization Policy Support
============================

OpenStack components are supposed to check user privileges to perform any
action. Generally these checks are role-based. See
http://docs.openstack.org/developer/keystone/architecture.html#approach-to-authorization-policy.
Zaqar needs to support policies as well.

Problem description
===================

Presently Zaqar is missing fine-grained permissions for actions. For example, it's
hard to allow one user only can get message from a queue but not be able to
post message to the queue, or something like that.


Proposed change
===============

Add policy check for all Zaqar API endpoints. This could be done by following
the same way as in other OpenStack components, by leveraging the oslo.policy
module which will do all the underlying work.

The implementation should be fairly simple, with oslo.policy's ``Enforcer``
class being instantiated with the policy file, then the ``enforce`` method used
to check each API call.

Proposed content of the policy file:

.. sourcecode:: json

    {
        "context_is_admin": "role:admin",
        "admin_or_owner":  "is_admin:True or project_id:%(project_id)s",
        "default": "rule:admin_or_owner",

        "queues:get_all": "",
        "queues:put": "",
        "queues:get": "",
        "queues:delete": "",

        "messages:post": "",
        "messages:get": "",
        "messages:bulk_get": "",
        "messages:bulk_delete": "",
        "messages:claim": "",

        "subscriptions:get_all": "",
        "subscriptions:create": "",
        "subscriptions:get": "",
        "subscriptions:delete": "",
        "subscriptions:update": "",
    }


Alternatives
------------

None.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Other end user impact
---------------------

None.

Deployer impact
---------------

An additional file, ``policy.json`` must be deployed. The deployer should
verify the settings in that file are correct for their deployment, such that
the correct users are allowed access.

Developer impact
----------------

If there is any new API endpoint added for Zaqar, then policy rules in the
json files should be updated accordingly.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  kragniz (Louis Taylor)

Milestones
----------

Target Milestone for completion: L-1

Work Items
----------

* Add config options to point to control policy file and settings
* Add policy check to all API calls
* Add unit tests
* Add documentation

Dependencies
============

* oslo.policy

Testing
=======

* Unit tests
* Manual testing

Documentation Impact
====================

* Feature need to be documented
* Add ``policy.json`` example
* Add documentation and examples of how to tweak policy settings

References
==========

* http://docs.openstack.org/developer/keystone/architecture.html#approach-to-authorization-policy
* http://docs.openstack.org/developer/keystone/api/keystone.openstack.common.policy.html
* http://docs.openstack.org/developer/keystone/configuration.html#keystone-api-protection-with-role-based-access-control-rbac
