..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===========================
 remove-pool-group-totally
===========================

https://blueprints.launchpad.net/zaqar/+spec/remove-pool-group-totally

remove-pool-group-totally.

Problem description
===================

Currently pool group is used in pool and flavor resource, but the pool group
only supports a 1:1 mapping with flavor. So it's not necessary to keep it
since we can map 1 flavor : N pool directly. Another issue is currently there
is no API to handle the pool group resource, which is very annoying to
maintain by operators.

For making a clarification to user, this bp proposes to remove useless
pool group from Zaqar, just keeps the pool resource.

For backward compatibility, we have split this work into two steps:

1. In Queens, we have supported the old way to use pool_group and the new way
   without it in Flavor both.

2. In Stein, we will remove the pool_group totally and only keep the new way
   in Flavor and Pool.

This bp will implement the second step and totally remove the pool group from
zaqar.


Proposed change
===============

1. Modify pool and flavor operation API:

   * Remove group from flavor operation API: like creat, update, show, list.
   * Remove group from pool operation API: like creat, update, show, list.

2. Remove the related logic codes about group in the zaqar server:

3. The Data model of flavor and pool resources have been changed in Queens,
   There is no need to discard pool_group field and Drop table PoolGroup
   we will just keep it in table for degrade convenience.

4. Modify the related logic codes about group in the zaqar client:

   * openstack messaging flavor create
   * openstack messaging flavor update
   * openstack messaging pool create
   * openstack messaging pool update


Drawbacks
---------
None

Alternatives
------------
None

Data model impact
-----------------
None

REST API impact
---------------

1. Create flavor API
The Request JSON when create flavor::

    PUT: /v2/flavors/{flavor_name}

    {
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
      "pool_list": [pool1, pool2, pool3]
   }

The response JSON when update flavor::


    {
      "href": "/v2/flavors/testflavor",
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
          "name": "test_flavor1",
          "pool_list": [pool1, pool2]
        },
        {
          "href": "/v2/flavors/test_flavor2",
          "name": "test_flavor2",
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
       "flavor": "testflavor1"
    }

The response JSON when update pools::


    {
      "href": "/v2/pools/test_pool",
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
          "flavor": "flavor1",
          "name": "test_pool1",
          "weight": 60,
          "uri": "mongodb://192.168.1.10:27017"
        },
        {
          "href": "/v2/pools/test_pool2",
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
flavor in the pool API. we will remove the old way in this bp.

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
  gecong<ge.cong@zte.com.cn>

Milestones
----------

Target Milestone for completion:
  Stein

Work Items
----------

#. Modify pool and flavor operation APIs.
#. Remove the related logic codes about group in the zaqar server.

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

