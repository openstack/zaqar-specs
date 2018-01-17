..
  This template should be in ReSTructured text. The filename in the git
  repository should match the launchpad URL, for example a URL of
  https://blueprints.launchpad.net/zaqar/+spec/awesome-thing should be named
  awesome-thing.rst.

  Please do not delete any of the sections in this
  template.  If you have nothing to say for a whole section, just write: None

  For help with syntax, see http://www.sphinx-doc.org/en/stable/rest.html
  To test out your formatting, see http://www.tele3.cz/jbar/rest/rest.html

===================
 Tests refactoring
===================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/zaqar/+spec/tests-refactoring

Zaqar tests are currently split into 2 different places, we want to reunite
them.

Problem description
===================

Tests currently exists in the tests top directory in the tree, but most of them
import abstract base classes from the zaqar.tests python package and set
attributes to be able to run them. There are also some tests definition in the
tests directory. It's not obvious that tests in the zaqar.tests package are not
run when using tox and in the gate, and it creates confusion on where the tests
should properly live.

Proposed change
===============

Move all the tests in the zaqar.tests package, and delete the tests directory.

To be able to have a reviewable stream, we will slowly add directories in
zaqar/tests to the testrepository configuration, and move them in small chunks.

Alternatives
------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  therve

Milestones
----------

Target Milestone for completion:
  Liberty-1

Work Items
----------

1. Add one tests directory to .testr.conf and move the tests over
2. Iterate over all the tests directories in zaqar/tests
3. Move remaining tests out of tests/

Dependencies
============

None.

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

