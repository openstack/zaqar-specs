..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

==============================
 Remove pools group from Zaqar
==============================

https://blueprints.launchpad.net/zaqar/+spec/remove-pool-group-from-zaqar

Remove pool group from pool and flavor resource.

Problem description
===================

Currently pool group is used in pool and flavor resource, but the pool group
only supports a 1:1 mapping with flavor. So it's not necessary to keep it
since we can map 1 flavor : N pool directly. Another issue is currently there
is no API to handle the pool group resource, which is very annoying to
maintain by operators.

For making a clarification to user, this bp proposes to remove useless
pool group from Zaqar, just keeps the pool resource.

For backward compatibility, we will split this work into two steps:

1. In Queens, we support the old way to use pool_group and the new way without
   it in Flavor both. The pool_group will be marked to let users know it will
   be useless soon and remove in R release.

2. In Rocky, we will remove the pool_group totally and only keep the new way
   in Flavor and Pool.


Proposed change
===============

1. Modify pool and flavor operation API:

   * Remove group from flavor operation API: like creat, update, show, list.
   * Remove group from pool operation API: like creat, update, show, list.

..note::

   This changes isn't going to happen in Queens release for backward
   compatibility, we keep the pool_group this circle and mark it will be
   useless in Zaqar log and remove the pool group totally in R release.


2. Modify the related logic codes about group in the zaqar server:

   For example, now in the creation of the queue, first we find all the pools
   base on group, and then select the appropriate pool (pool = select.weighted
   (pools)) to create the queue.

   We modify as follows:
   Find all the pools base on flavor, and then select the appropriate pool to
   create a queue.

   etc.

3. Change the Data model of flavor and pool resources, detail see Data model
   impact.

4. Modify the related logic codes about group in the zaqar client:

   * openstack messaging flavor create
   * openstack messaging flavor update
   * openstack messaging pool create
   * openstack messaging pool update

..note::

   Like the changes in Zaqar APIs, this change is also going to do in R
   release.


Drawbacks
---------
None

Alternatives
------------
None

Data model impact
-----------------
For sqlalchemy:

* Drop table PoolGroup in R.

* Modify Flavors table:

    Discard pool_group field, but do not remove the field in Q in order to be
        compatible with the latter :
        CREATE TABLE `Flavors` (
        `name` varchar(64) NOT NULL,
        `project` varchar(64) DEFAULT NULL,
        `pool_group` varchar(64) NOT NULL,
        `capabilities` text,
        PRIMARY KEY (`name`),
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

* Modify Pools table:

    Discard group field, but do not remove the field in Q in order to be
        compatible with the latter;
        Add flavor field, this flavor field equals name field of the flavor:
        CREATE TABLE `Pools` (
        `name` varchar(64) NOT NULL,
        `group` varchar(64) DEFAULT NULL,
        `uri` varchar(255) NOT NULL,
        `weight` int(11) NOT NULL,
        `options` text,
        `flavor` varchar(64) DEFAULT NULL,
        PRIMARY KEY (`name`),
        UNIQUE KEY `uri` (`uri`),
        CONSTRAINT `Pools_flavor_fk` FOREIGN KEY (`flavor`)
        REFERENCES `fla` (`name`) ON DELETE CASCADE
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

* Migration strategy for Flavor and Pools table:

   #. Query all the Pools, and then process every record as follows:
   #. Find Flavors.name where Flavors.pool_group = Pools.group
   #. Modify Pools.flavor = Flavors.name

2. For mongodb:

* The upgrade strategy is similar. We need to introduce a script to implement
  the data migration.

..note::

   Since we decide to remove the pool_group totally in Rocky, that then we need
   to provide a script to help users to migrate the data and remove the
   pool_group column from db table.


REST API impact
---------------

1. Create flavor API
The Request JSON when create flavor::

    PUT: /v2/flavors/{flavor_name}

    {
      "pool_group": "testgroup",  # remove pool_group in Rocky release
      "pool_list": [pool1, pool2]
    }

The response JSON when Create flavor::

    Normal response codes: 201

    Error response codes:
    •BadRequest (400)
    •Unauthorized (401)
    •Forbidden (403)

2. Update flavor API
The Request JSON when update flavor::

    PATCH: /v2/flavors/{flavor_name}

   {
      "pool_group": "testgroup",  # remove pool_group in Rocky release
      "pool_list": [pool1, pool2, pool3]
   }

The response JSON when update flavor::


    {
      "href": "/v2/flavors/testflavor",
      "pool_group": "testgroup",# remove pool_group in Rocky release
      "name": "testflavor",
      "capabilities": [
        "FIFO",
        "CLAIMS",
        "DURABILITY",
        "AOD",
        "HIGH_THROUGHPUT"
      ],
      "pool_list": [pool1, pool2, pool3]
    }
     Normal response codes: 200

     Error response codes:
     •BadRequest (400)
     •Unauthorized (401)
     •Forbidden (403)
     •Not Found (404)
     •ServiceUnavailable (503)

3. Shows details for a flavor API
The response JSON when show details flavor::

    GET: /v2/flavors/{flavor_name}

    {
      "href": "/v2/flavors/testflavor",
      "pool_group": "testgroup", # remove pool_group in Rocky release
      "capabilities": [
        "FIFO",
        "CLAIMS",
        "DURABILITY",
        "AOD",
        "HIGH_THROUGHPUT"
      ],
      "pool_list": [pool1, pool2],
      "name": "testflavor"
    }

The response JSON when show details flavor::

     Normal response codes: 200

     Error response codes:
     •BadRequest (400)
     •Unauthorized (401)
     •Forbidden (403)
     •Not Found (404)
     •ServiceUnavailable (503)

4. List flavor API
The response JSON when list flavors::

    GET: /v2/flavors

    {
      "flavors": [
        {
          "href": "/v2/flavors/test_flavor1",
          "pool_group": "testgroup", # remove pool_group in Rocky release
          "name": "test_flavor1",
          "pool": "testgroup",       # remove pool_group in Rocky release
          "pool_list": [pool1, pool2]
        },
        {
          "href": "/v2/flavors/test_flavor2",
          "pool_group": "testgroup", # remove pool_group in Rocky release
          "name": "test_flavor2",
          "pool": "testgroup",       # remove pool_group in Rocky release
          "pool_list": [pool3, pool4]
        }
      ],
      "links": [
        {
          "href": "/v2/flavors?marker=test_flavor2",
          "rel": "next"
        }
      ]
    }

The response JSON when list flavors::

     Normal response codes: 200

     Error response codes:
     •Unauthorized (401)
     •Forbidden (403)


5. Create pools API
The Request JSON when create pools::

     PUT: /v2/pools/{pool_name}

    {
        "weight": 100,
        "uri": "mongodb://127.0.0.1:27017",
        "options":{
            "max_retry_sleep": 1
        },
        "group": "poolgroup", # remove pool_group in Rocky release
        "flavor": "testflavor"
    }


The response JSON when Create pools::

    Normal response codes: 201

    Error response codes:
    •BadRequest (400)
    •Unauthorized (401)
    ••Conflict (409)

6. Update pools API
The Request JSON when update pools::

    PATCH: /v2/pools/{pool_name}

    {
       "weight": 60,
       "uri": "mongodb://127.0.0.1:27017",
       "options":{
           "max_retry_sleep": 1
       },
       "group": "newpoolgroup", # remove pool_group in Rocky release
       "flavor": "testflavor1"
    }

The response JSON when update pools::


    {
      "href": "/v2/pools/test_pool",
      "group": "newpoolgroup", # remove pool_group in Rocky release
      "name": "test_pool",
      "weight": 60,
      "uri": "mongodb://127.0.0.1:27017",
      "flavor": "testflavor1"
    }
     Normal response codes: 200

     Error response codes:
     •BadRequest (400)
     •Unauthorized (401)
     •Not Found (404)
     •ServiceUnavailable (503)

7. Shows details for a pool API
The response JSON when show details pool::

    GET: /v2/pools/{pool_name}

    {
      "href": "/v2/pools/test_pool",
      "group": "testpoolgroup", # remove pool_group in Rocky release
      "flavor": "flavor1",
      "name": "test_pool",
      "weight": 100,
      "uri": "mongodb://127.0.0.1:27017"
    }

The response JSON when show details pool::

     Normal response codes: 200

     Error response codes:
     •BadRequest (400)
     •Unauthorized (401)
     •ServiceUnavailable (503)

8. List pools API
The response JSON when list pools::

    GET: /v2/pools

    {
      "pools": [
        {
          "href": "/v2/pools/test_pool1",
          "group": "poolgroup", # remove pool_group in Rocky release
          "flavor": "flavor1",
          "name": "test_pool1",
          "weight": 60,
          "uri": "mongodb://192.168.1.10:27017"
        },
        {
          "href": "/v2/pools/test_pool2",
          "group": "poolgroup", # remove pool_group in Rocky release
          "flavor": "flavor1",
          "name": "test_pool2",
          "weight": 40,
          "uri": "mongodb://192.168.1.20:27017"
        }
      ],
      "links": [
        {
          "href": "/v2/pools?marker=test_pool2",
          "rel": "next"
        }
      ]
    }

The response JSON when list pools::

     Normal response codes: 200

     Error response codes:
     •Unauthorized (401)
     •Not Found (404)


We use the v2 interface, just add pool_list in the flavor API and add the
flavor in the pool API. This can be compatible with the old way.

#. Old Way:

   * configure group in pool API and in flavor API;

#. New Way:

   * configure pool_list in flavor API
   * add a pool to flavor:

      #. method one: update pool_list in flavor API
      #. method two: config a pool with the flavor in pool API.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao<sxmatch1986@gmail.com>
Secondary assignee:
  gengchc2 <geng.changcai2@zte.com.cn>

Milestones
----------

Target Milestone for completion:
  Queens and Rocky

Work Items
----------

#. Modify pool and flavor operation APIs.
#. Modify the related logic codes about group in the zaqar server.
#. Modify the Data model about Flavors , Pools and PoolGroup.
#. Modify the related logic codes about group in the zaqar client.

Dependencies
============

None

Testing
=======

Both unit and Tempest tests need to be created to cover the code change.


Documentation Impact
====================

The Zaqar API documentation will need to be updated to reflect the REST
API changes.

References
==========
None

