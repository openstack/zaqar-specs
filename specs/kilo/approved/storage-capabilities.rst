..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://sphinx-doc.org/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

=============================
 Expose storage capabilities
=============================

Flavors allow operators to expose some of the internal storage capabilities without giving the exact details of how the system is configured. As of now, the operators has total control over what capabilities are exposed to the user regardeless on whether these capabilities are indeed guaranteed or valid. A `capability` is a feature that is either specific to the storage or to Zaqar - durable, fifo, in-memory, for example - and they can be enabled/exposed in a per-flavor basis allowing end-users to choose the best flavor for a specific queue.

https://blueprints.launchpad.net/zaqar/+spec/expose-storage-capabilities


Problem description
===================

Flavors are very powerful when it comes to give choices to end-users without taking control out from operators. However, there's no way for operators to know what capabilities each storage driver has, which makes this feature completely custom to whatever the operator thinks makes more sense.

It's important to support operators on the choice of what features are exposed to the end-users through the API by making it clear which are supported by the drivers that have been deployed. Moreover, this is also important for doing a proper capacity planning based on the use-cases the service is trying to support.

Proposed change
===============

Capabilities
------------

This change request proposes defining a set of internal `capabilities` and a way for storage drivers to expose which capabilities are supported so that operators may know in advance what a flavor would support and what storage drivers they may need.

The set of internal capabilities includes:

- FIFO
- AOD (At least once delivery)
- high-throughput
- claims

Exposing Capabilities
---------------------

This capabilities should be exposed by the storage driver iff supported. This spec proposes adding a new class method to the base class called `capabilities` that should return a set with a list of supported `capabilities`. The capabilities returned in this set cannot be custom, which means they must be supported internally.::

    @six.add_metaclass(abc.ABCMeta)
    class DataDriverBase(DriverBase):
        ...

    @property
    def capabilities(self):
        return self._CAPABILITIES


Drivers Load
------------

In addition to the above-mentioned changes, we also need a better way to load drivers based on the capabilities. That is, when drivers are loaded, it's necessary to pass down to the driver the capabilities that have been enabled per-flavor so that the driver itself can be specialized for some capabilities. In most of the cases, loading a driver will end up in always loading the same implementation. However, there are cases - FIFO, for example - where more specialized implementations may be needed depending on the storage. Therefore, instead of *always* loading the `DataDriver`, this spec proposes adding a new `get_driver` function that will let the storage itself load a driver according to the capabilities passed there.::

    def get_driver(plane='data', capabilities=None):
        ...


This means we won't need to register neither `DataDriver`s nor `ControlDriver`s as `entry_points` but just the new `get_driver` function. This will allow storage driver's to expose fewer things as public API's. A variant for the above `get_driver` proposal would be.::

  def get_driver(base=base.BaseDataDriver, capabilities=None):
      ...

This way we can avoid giving 'names' to this planes and just ask what we need in terms of 'signature'. However, it makes loading the actual driver a bit more complex and it'll also duplicate some logic across different storage drivers.

Pools and Flavors
-----------------

Flavors have support for capabilities. As of now, these capabilities
are 100% custom. With the implementation of this spec, it'll be
possible for flavors to know in advance what capabilities are
supported based on the stores registered in the pool.

Pools, however, shouldn't allow storage drivers with different
capabilities to co-exist under the same pool. That is to say that
pools will have to verify the storage capabilities when new nodes are
added to the pool and ensure that the capabilities supported by the
new node are consistent with the rest of the nodes in the pool.

Alternatives
------------

Keep it custom and do nothing.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  flaper87

Secondary assignee:
  vkmc

Milestones
----------

Target Milestone for completion:
  K-1

Work Items
----------

* Define list of internal capabilities and document it
* Add required abstractions to storage drivers to expose the supported capabilities
* Let flavor's pass the capabilities down to the storage driver when it's being loaded.

Dependencies
============

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

