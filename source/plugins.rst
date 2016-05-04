.. _plugins:

==============
TokuDB Plugins
==============

Plugins enable you to extend the functionality of the TokuDB storage engine.

.. contents::
  :local:
  :depth: 1

To install plugins, use the ``INSTALL PLUGIN`` command. To verify which plugins are available, use the ``SHOW PLUGINS`` command.

tokudb_file_map
---------------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_file_map`` table, which contains all Fractal Tree indexes and their corresponding data files.

tokudb_fractal_tree_info
------------------------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_fractal_tree_info`` table, which contains Fractal Tree internals used primarily for development and debugging by Percona staff.

tokudb_fractal_tree_block_map
-----------------------------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_fractal_tree_block_map`` table, which contains block file information (the file format behind Fractal Trees) used primarily for development and debugging by Percona staff.

tokudb_lock_waits
-----------------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_lock_waits`` table, which contains the set of lock requests not granted due to a lock conflict with some other transaction.

tokudb_locks
------------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_locks`` table, which contains the set of locks granted to TokuDB transactions.

tokudb_trx
----------

This plugin exposes the ``INFORMATION_SCHEMA.tokudb_trx`` table, which maps TokuDB transaction identifiers to MySQL client identifiers. This mapping enables you to associate a TokuDB transaction with a MySQL client operation.

For more information about lock visualization, see :ref:`lock-visual`.
