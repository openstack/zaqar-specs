..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=====================================
Remove format constraint of client id
=====================================

https://blueprints.launchpad.net/zaqar/+spec/remove-format-constraint-of-client-id

Since some clients use different format of client id not only uuid,
so Zaqar will support this function. This also requires user to ensure the
client id is immutable.


Problem description
===================

Now Zaqar has format constraint to client id, must be UUID format. But this
constraint is not very reasonable under some cases, like user want to use
user id in LDAP as client id, but Zaqar doesn't allow this now.


Proposed change
===============
* Add Three config options:

  #. 'client_id_uuid_safe': Defines the format of client id, the value could be
     "strict" or "off". "strict" means the format of client id must be uuid,
     "off" means the restriction be removed. The default value is 'strict'.
  #. 'min_length_client_id': Defines the minimum length of client id if remove
     the uuid restriction. Default value is 10.
  #. 'max_length_client_id': Defines the maximum length of client id if
     remove the uuid restriction. Default value is 36.

* Will change the method 'require_client_id' in wsgi/helpers.py to support
  validating the different format of client id.

API Impact
-----------
For WSGI: Will impact all APIs in v1.1 and v2 version.

For WebSocket: Will impact message list and message post in v2 version. And we
will support others APIs in following patch.

Drawbacks
---------
There may be a tiny performance impact when user uses very long client id since
Zaqar need to store the client id in storage layer.


Alternatives
------------

N/A


Implementation
==============

Assignee(s)
-----------

wanghao<sxmatch1986@gmail.com>

Milestones
----------

Rocky-3

Work Items
----------

#* Change the Zaqar server code.
#* Update API reference.
#* Add release note for this feature.
#* UTs for this feature.

Dependencies
============

N/A
