..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=========================================
Introduce Topic Resource For Notification
=========================================

https://blueprints.launchpad.net/zaqar/+spec/introduce-topic-resource-for-notification

We want to introduce a new resource called Topic into Zaqar.
Topic is a concept from AWS Simple Notification Service (SNS), it will has
relevance with subscriptions. User can send message to a topic,
and then the subscribers will get the message according to different protocols,
like http, email, sms, etc.

Problem description
===================

Now basically Zaqar has integrated two types of services,
Messaging Queue Service and Notification Service. We create subscriptions
based on queue, and also introduce retry policy for subscription and queue.
It's not very clear from the service view, especially for the users who have
been used to the SNS and SQS in AWS

So in Rocky, we want to introduce the Topic resource into Zaqar, and split
Messaging Queue Service and Notification Service clearly.

AWS SNS service has the resource named Topic, that would be a good reference
for Zaqar[1].

Proposed change
===============

To implement the Topic resource, one idea is to define the Topic as a special
queue with some special feature that is different from common queue.

1. Could set the users or projects who can send topic message to this Topic

2. Could set the users or projects who can subscribe this Topic.

3. The messages will be stored in Topic but those messages could NOT be claimed
   since they are just used for sending to subscribers. When sending
   successfully, Zaqar will delete this message from topic, if not, will retry
   the sending process.

Besides those features, user also can set the retry policy to Topic and
Subscription both.

After introducing Topic to Zaqar, we will keep the subscription of common
queue for a while, but we also want to change the users' behavior to use Topic
if they want to subscribe something in Zaqar. So maybe we remove the
subscription of common queue in future.

API Impact
----------

1. Topic Creation::

    PUT: /v2/topics/{topic_name}
    BODY:
    {
       "_allowed_sending_users": <List of user id, optional>,
       "_allowed_sending_projects": <List of project id, optional>,
       "_allowed_subscribe_users": <List of user id, optional>,
       "_allowed_subscribe_projects": <List of project id, optional>,
       "_allowed_subscribe_protocols": <List of string, optional>,
       "_retry_policy": <Dict, optional>
    }
    RESPONSE CODES: 201, 204
    ERROR RESPONSE CODES:
    * BadRequest (400)
    * Unauthorized (401)
    * ServiceUnavailable (503)

.. note::

  "_allowed_sending_users": define which users can send message to topic.
  "_allowed_sending_projects": define which projects can send message to topic.
  Of course you can set the project owned this topic here too.
  "_allowed_subscribe_users": define which users can subscribe topic.
  "_allowed_subscribe_projects": define which projects can subscribe topic.
  The default behavior is allowing all the users under the project who create
  this topic send message, and all projects could subscribe this topic.
  Default subscribe protocols are email, webhook, and trust.
  The retry policy is same as Queue's retry policy.

2. Topic Update::

    PATCH: /v2/topics/{topic_name}
    BODY:
    [
       {
          "op": "replace",
          "path": "/metadata/allowed_sending_projects",
          "value": [pro_id1, pro_id2, ... ,pro_idN]
       }
    ]
    RESPONSE CODE: 200
    ERROR RESPONSE CODES:
    * BadRequest (400)
    * Unauthorized (401)
    * Not Found (404)
    * Conflict (409)
    * ServiceUnavailable (503)

3. Topic Query List::

    GET: /v2/topics
    RESPONSE BODY:
    {
       "topics": [
                    {
                       "topic_name": xxx,
                       "href": xxx
                    }
                 ],
       "links": [
                   {
                       "href": '/v2/topic?marker=wellington',
                       "rel": "next"
                   }
                ]
    }
    RESPONSE CODE: 200
    ERROR RESPONSE CODES:
    * BadRequest (400)
    * Unauthorized (401)
    * ServiceUnavailable (503)

4. Topic Query Detail::

    GET: /v2/topics/{topic_name}
    RESPONSE BODY:
    {
       "_allowed_sending_users": [user_id1, ... ,user_idN],
       "_allowed_sending_projects": [pro_id1, ... ,pro_idN],
       "_allowed_subscribe_users": [user_id1, ... ,user_idN],
       "_allowed_subscribe_projects": [pro_id1, ... ,pro_idN],
       "_allowed_subscribe_protocols": [email, webhook, trust],
       "_retry_policy": {}
    }
    RESPONSE CODE: 200
    ERROR RESPONSE CODES:
    * BadRequest (400)
    * Unauthorized (401)
    * ServiceUnavailable (503)

5. Topic Delete::

    DELETE: /v2/topics/{topic_name}
    RESPONSE CODE: 204
    ERROR RESPONSE CODES:
    * BadRequest (400)
    * Unauthorized (401)
    * ServiceUnavailable (503)

Drawbacks
---------

N/A

Alternatives
------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao (sxmatch1986@gmail.com)

Milestones
----------

R-3

Work Items
----------

* Implement the Topic API.
* Implement the process of CRUD.
* UTs for this feature.
* DOC support.

Dependencies
============

[1]: http://docs.aws.amazon.com/sns/latest/dg/welcome.html
