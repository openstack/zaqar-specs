..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

======================================
Support redis as mgmt storage backend
======================================

https://blueprints.launchpad.net/zaqar/+spec/support-redis-as-management-storage-backend


Redis is different than other database solutions in many ways, it support
two modes: memory storage and disk storage, We can select one of the two modes
by modifying redis.conf. If we add save options in redis.conf, redis can
cyclically save the data into disk:
save 900 1
save 300 10
save 60 10000

Redis includes a fairly high performance implementation of database.
Currently, zaqar supports redis, mongodb and swift as the storage backend.
After performance testing, we find that the performance of redis
is the highest among these storage-backend. Using Redis, we may provide a
system which is very practical to use, easy to administer and scale, while
providing excellent performances.


Problem description
===================

In order to enhance the performance of zaqar, it is necessary to support
redis as a management database backend.

At present, Zaqar has supported sqlalchemy and mongodb as management storage
backend, that would be a good reference for redis.



Proposed change
===============

1. Add CatalogueController in redis storage:

  Add such functions: insert, update, delete, list, get, drop_all etc.


2. Add FlavorsController in redis storage:

  Add such functions: insert, update, delete, list, get, drop_all etc.

3. Add PoolsController in redis storage:

  Add such functions: insert, update, delete, list, get, drop_all etc.


Drawbacks
---------
None

Alternatives
------------
MongoDB SQL

Data model impact
-----------------

We need to add three table models, as follows:

1. Redis Data Structures for table catalogue:

* All Project (Redis sorted set):

        Set of all queue_ids, ordered by name. Used to delete the all
        records of table catalogue.

        Key: catalogue

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <project_id>.<queue_name>  |
        +--------+-----------------------------+

* Project Index (Redis sorted set):

        Set of all queue_ids for the given project, ordered by name.

        Key: <project_id>.catalogue

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <project_id>.<queue_name>  |
        +--------+-----------------------------+

* Queue and pool Information (Redis hash):

        Key: <project_id>.<queue_name>.catalogue

        +----------------------+---------+
        |  Name                |  Field  |
        +======================+=========+
        |  Project             |  p      |
        +----------------------+---------+
        |  Queue               |  p_q    |
        +----------------------+---------+
        |  Pool                |  p_p    |
        +----------------------+---------+

2. Redis Data Structures for table flavor:

* All flavor_ids (Redis sorted set):

        Set of all flavor_ids, ordered by name. Used to delete the all
        records of table flavors.

        Key: flavors

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <flavor>                   |
        +--------+-----------------------------+

* Project Index (Redis sorted set):

        Set of all flavors for the given project, ordered by name.

        Key: <project_id>.flavors

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <flavor>                   |
        +--------+-----------------------------+

* Flavor Information (Redis hash):

        Key: <flavor_id>.flavors

        +----------------------+---------+
        |  Name                |  Field  |
        +======================+=========+
        |  flavor              |  f      |
        +----------------------+---------+
        |  project             |  p      |
        +----------------------+---------+
        |  capabilities        |  c      |
        +----------------------+---------+

3. Redis Data Structures for table pools:

* All pool (Redis sorted set):

        Set of all pool_ids, ordered by name. Used to delete the all
        records of table pools.

        Key: pools

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <pool>                     |
        +--------+-----------------------------+

* Flavor Index (Redis sorted set):

        Set of all pool_ids for the given flavor, ordered by name.

        Key: <flavor>.pools

        +--------+-----------------------------+
        |  Id    |  Value                      |
        +========+=============================+
        |  name  |  <pool>                     |
        +--------+-----------------------------+

* Pools Information (Redis hash):

        Key: <pool>.pools

        +----------------------+---------+
        |  Name                |  Field  |
        +======================+=========+
        |  pool                |  p      |
        +----------------------+---------+
        |  uri                 |  u      |
        +----------------------+---------+
        |  weight              |  w      |
        +----------------------+---------+
        |  options             |  o      |
        +----------------------+---------+


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  gengchc2 <geng.changcai2@zte.com.cn>

Secondary assignee:
  gecong <ge.cong@zte.com.cn>

Milestones
----------

Target Milestone for completion:
  Queens-Q2

Work Items
----------

#. Add CatalogueController in redis storage.
#. Add FlavorsController in redis storage.
#. Add PoolsController in redis storage.

Dependencies
============

[1]:https://review.opendev.org/#/c/345133/

Testing
=======

Both unit and Tempest tests need to be created to cover the code change.


Documentation Impact
====================

#. Add docs about the configuration of redis and HA.

References
==========
None

[2]:http://paste.openstack.org/show/621298/

