..
     This work is aimed to implement zaqar email delivery
 through smtp service,and Chinese translation support.

Support zaqar_sendmail
==================================

https://blueprints.launchpad.net/zaqar/+spec/zaqar-email-delivery

Problem description
===================

Currently the email subscription in Zaqar relay on the third part tools, such
as "sendmail". It means that deployer should install it out of Zaqar. If he
forgets, Zaqar will raise internal error. This work let Zaqar support email
subscription by itself using the ``smtp`` python library.

Use Cases
---------

The modification enables users to send email subscriptions without the third
part tools.

Proposed change
===============

A new config option ``smtp_mode`` will be added, it has two value choices,
["third_part", "self_local"]. ``third_part`` means Zaqar will use the tools
from which ``smtp_command`` config option provides to send email subscription.
``self_local`` means Zaqar will use the ``smtp`` python library instead.

A email template file will be added as well. Some parameters in the file should
be configured firstly before using. Those parameters include something like
smtp service account/passwrod, smtp service address and so on. These value
should come from config options as well.

The email body in the template file should be configured as well. Currently,
it can only be managed by operators out of Zaqar. In the future, we can use
jinja template way to let it be configurable by Zaqar.

Once ``self_local`` is used, Zaqar will fill up the template file first, then
use ``stmp`` lib to send the email subscription.

Alternatives
------------

None


REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None


Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

A new config option called ``smtp_mode`` is added. To keep backward
compatibility the default behavior is keeping use third part tools to send
email subscription. If you don't want to install the third part email tools
anymore, please change its value to ``self_local``.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

changyufei<changyufei@unitedstack.com>

Work Items
----------

1. Add a new config option ``smtp_mode``
2. Add the email template
3. Add the logic to use ``smtp`` python lib
4. The test code should be added.
5. Document.

Milestones
----------

R-3


Dependencies
============

None

Testing
=======

For functional test, here is no good way to test it in Upstream since
OpenStack CI/CD environment doesn't has a email service. Or we may ask the
community for help to give Zaqar team a test account within @openstack.org
email system. Or any other third part company can provide one for us. Otherwise
we could only test it locally to make sure it works well.


Documentation Impact
====================

A new send email way will be added. It should be documented.

References
==========

None
