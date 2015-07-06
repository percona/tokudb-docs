.. highlight:: mysql

.. _using-tokudb:

============
Using TokuDB
============

 .. caution:: Do not move or modify any TokuDB files. You will break the database, and need to recover the database from a backup.
 
Fast Insertions and Richer Indexes
----------------------------------

TokuDB's fast indexing enables fast queries through the use of rich indexes, such as covering and clustering indexes. It's worth investing some time to optimize index definitions to get the best performance from MySQL and TokuDB. Here are some resources to get you started:

* "Understanding Indexing" by Zardosht Kasheff (`video <http://vimeo.com/26454091>`_) (`slides <http://www.tokutek.com/wp-content/uploads/2011/04/understanding_indexing_MySQL_UC.pdf>`_)

* `Rule of Thumb for Choosing Column Order in Indexes <http://www.mysqlperformanceblog.com/2009/06/05/a-rule-of-thumb-for-choosing-column-order-in-indexes/>`_

* `TokuView Blog <http://www.tokutek.com/tokuview>`_, the Tokutek Blog has many entries on indexing. Please see these specific examples and more:

  * `Covering Indexes: Orders-of-Magnitude Improvements <http://tokutek.com/2009/05/covering_indexes_orders_of_magnitude_improvements/>`_

  * `Introducing Multiple Clustering Indexes <http://tokutek.com/2009/05/introducing_multiple_clustering_indexes/>`_

  * `Clustering Indexes vs. Covering Indexes <http://tokutek.com/2009/05/clustering_indexes_vs_covering_indexes/>`_

  * `How Clustering Indexes Sometimes Helps UPDATE and DELETE Performance <http://tokutek.com/2009/06/how_clustering_indexes_sometimes_help_update_and_delete_performance/>`_

* *High Performance MySQL, 3rd Edition* by Baron Schwartz, Peter Zaitsev, Vadim Tkachenko, Copyright 2012, O'Reilly Media. See Chapter 5, *Indexing for High Performance*.

Clustering Secondary Indexes
----------------------------

One of the keys to exploiting TokuDB's strength in indexing is to make use of clustering secondary indexes.

To define a secondary index as clustering, simply add the word ``CLUSTERING`` before the key definition. For example::

 CREATE TABLE table (
   column_a INT,
   column_b INT,
   column_c INT,
   PRIMARY KEY index_a (column_a),
   CLUSTERING KEY index_b (column_b)) ENGINE = TokuDB;

In the previous example, the primary table is indexed on *column_a*. Additionally, there is a secondary clustering index (named *index_b*) sorted on *column_b*. Unlike non-clustered indexes, clustering indexes include all the columns of a table and can be used as covering indexes. For example, the following query will run very fast using the clustering *index_b*::

 SELECT column_c
   FROM table
   WHERE column_b BETWEEN 10 AND 100;

This index is sorted on *column_b*, making the ``WHERE`` clause fast, and includes *column_c*, which avoids lookups in the primary table to satisfy the query.

TokuDB makes clustering indexes feasible because of its excellent compression and very high indexing rates. For more information about using clustering indexes, see `Introducing Multiple Clustering Indexes <http://tokutek.com/2009/05/introducing_multiple_clustering_indexes/>`_

Hot Index Creation
------------------

TokuDB enables you to add indexes to an existing table and still perform inserts and queries on that table while the index is being created.

The ``ONLINE`` keyword is not used. Instead, the value of the ``tokudb_create_index_online`` client session variable is examined. More information is available in :ref:`TokuDB Variables <tokudb_create_index_online>`.

Hot index creation is invoked using the ``CREATE INDEX`` command after setting ``tokudb_create_index_online=on`` as follows:

.. code-block:: mysql
   :emphasize-lines: 1,5

   SET tokudb_create_index_online=ON;
 
   Query OK, 0 rows affected (0.00 sec)
 
   CREATE INDEX index ON table (field_name);

Alternatively, using the ``ALTER TABLE`` command for creating an index will create the index offline (with the table unavailable for inserts or queries), regardless of the value of ``tokudb_create_index_online``. The only way to hot create an index is to use the ``CREATE INDEX`` command.

Hot creating an index will be slower than creating the index offline, and progress depends how busy the mysqld server is with other tasks. Progress of the index creation can be seen by using the ``SHOW PROCESSLIST`` command (in another client). Once the index creation completes, the new index will be used in future query plans.

If more than one hot ``CREATE INDEX`` is issued for a particular table, the indexes will be created serially. An index creation that is waiting for another to complete will be shown as *Locked* in ``SHOW PROCESSLIST``. We recommend that each ``CREATE INDEX`` be allowed to complete before the next one is started.

Hot Column Add, Delete, Expand, and Rename (HCADER)
---------------------------------------------------

TokuDB enables you to add or delete columns in an existing table, expand char, varchar, varbinary, and integer type columns in an existing table, or rename an existing column in a table with little blocking of other updates and queries. HCADER typically blocks other queries with a table lock for no more than a few seconds. After that initial short-term table locking, the system modifies each row (when adding, deleting, or expanding columns) later, when the row is next brought into main memory from disk. For column rename, all the work is done during the seconds of downtime. On-disk rows need not be modified.

To get good performance from HCADER, observe the following guidelines.

* The work of altering the table for column addition, deletion, or expansion is performed as subsequent operations touch parts of the Fractal Tree, both in the primary index and secondary indexes.

  You can force the column addition, deletion, or expansion work to be performed all at once using the standard syntax of ``OPTIMIZE TABLE X``, when a column has been added to, deleted from, or expanded in table X. It is important to note that as of TokuDB version 7.1.0, ``OPTIMIZE TABLE`` is also hot, so that a table supports updates and queries without blocking while an ``OPTIMIZE TABLE`` is being performed. Also, a hot ``OPTIMIZE TABLE`` does not rebuild the indexes, since TokuDB indexes do not age. Rather, they flush all background work, such as that induced by a hot column addition, deletion, or expansion.

* Each hot column addition, deletion, or expansion operation must be performed individually (with its own SQL statement). If you want to add, delete, or expand multiple columns use multiple statements.

* Avoid adding, deleting, or expanding a column at the same time as adding or dropping an index.

* The time that the table lock is held can vary. The table-locking time for HCADER is dominated by the time it takes to flush dirty pages, because MySQL closes the table after altering it. If a checkpoint has happened recently, this operation is fast (on the order of seconds). However, if the table has many dirty pages, then the flushing stage can take on the order of minutes.

* Avoid dropping a column that is part of an index. If a column to be dropped is part of an index, then dropping that column is slow. To drop a column that is part of an index, first drop the indexes that reference the column in one alter table statement, and then drop the column in another statement.

* Hot column expansion operations are only supported to char, varchar, varbinary, and integer data types. Hot column expansion is not supported if the given column is part of the primary key or any secondary keys.

* Rename only one column per statement. Renaming more than one column will revert to the standard MySQL blocking behavior. The proper syntax is as follows::

 ALTER TABLE table
   CHANGE column_old column_new
   DATA_TYPE REQUIRED_NESS DEFAULT

Here's an example of how that might look::

 ALTER TABLE table
   CHANGE column_old column_new 
   INT(10) NOT NULL;

Notice that all of the column attributes must be specified. ``ALTER TABLE table CHANGE column_old column_new;`` induces a slow, blocking column rename.

* Hot column rename does not support the following data types: ``TIME``, ``ENUM``, ``BLOB``, ``TINYBLOB``, ``MEDIUMBLOB``, ``LONGBLOB``. Renaming columns of these types will revert to the standard MySQL blocking behavior.

* Temporary tables cannot take advantage of HCADER. Temporary tables are typically small anyway, so altering them using the standard method is usually fast.

Compression Details
-------------------

TokuDB offers different levels of compression, which trade off between the amount of CPU used and the compression achieved. Standard compression uses less CPU but generally compresses at a lower level, high compression uses more CPU and generally compresses at a higher level. We have seen compression up to 25x on customer data.

Compression in TokuDB occurs on background threads, which means that high compression need not slow down your database. Indeed, in some settings, we've seen higher overall database performance with high compression.

.. note:: We recommend that users use standard compression on machines with six or fewer cores, and high compression on machines with more than six cores.

The ultimate choice depends on the particulars of how a database is used, and we recommend that users use the default settings unless they have profiled their system with high compression in place.

Compression is set on a per-table basis and is controlled by setting row format during a ``CREATE TABLE`` or ``ALTER TABLE``. For example::

 CREATE TABLE table (
   column_a INT NOT NULL PRIMARY KEY,
   column_b INT NOT NULL) ENGINE=TokuDB
   ROW_FORMAT=row_format;

If no row format is specified in a ``CREATE TABLE``, the table is compressed using whichever row format is specified in the session variable ``tokudb_row_format``. If no row format is set nor is ``tokudb_row_format``, the zlib compressor is used.

``row_format`` and ``tokudb_row_format`` variables accept the following values:

* ``tokudb_default``: This sets the compression to the default behavior. As of TokuDB 7.1.0, the default behavior is to compress using the zlib library. In the future this behavior may change.

* ``tokudb_fast``: This sets the compression to use the quicklz library.

* ``tokudb_small``: This sets the compression to use the lzma library.

In addition, you can choose a compression library directly, which will override previous values. The following libraries are available:

* ``tokudb_zlib``: Compress using the zlib library, which provides mid-range compression and CPU utilization.

* ``tokudb_quicklz``: Compress using the quicklz library, which provides light compression and low CPU utilization.

* ``tokudb_lzma``: Compress using the lzma library, which provides the highest compression and high CPU utilization.

* ``tokudb_uncompressed``: This setting turns off compression and is useful for tables with data that cannot be compressed.

Changing Compression of a Table
-------------------------------

Modify the compression used on a particular table with the following command::

 ALTER TABLE table
   ROW_FORMAT=row_format;

.. note:: Changing the compression of a table only affects newly written data (dirtied blocks). After changing a table's compression you can run ``OPTIMZE TABLE`` to rewrite all blocks of the table and its indexes.

.. _read-free-repl:

Read Free Replication
---------------------

TokuDB slaves can be configured to perform significantly less read IO in order to apply changes from the master. By utilizing the power of Fractal Tree indexes:

* insert/update/delete operations can be configured to eliminate read-modify-write behavior and simply inject messages into the appropriate Fractal Tree indexes

* update/delete operations can be configured to eliminate the IO required for uniqueness checking

To enable Read Free Replication, the servers must be configured as follows:

* On the replication master:

  * Enable row based replication: set ``BINLOG_FORMAT=ROW``

* On the replication slave(s):

  * The slave must be in read-only mode: set ``read_only=1``

  * Disable unique checks: set ``tokudb_rpl_unique_checks=0``

  * Disable lookups (read-modify-write): set ``tokudb_rpl_lookup_rows=0``

.. note:: You can modify one or both behaviors on the slave(s).

.. note:: In MySQL 5.5 and MariaDB 5,5, only tables with a defined primary key are eligible for this optimization. The limitation does not apply to MySQL 5.6, Percona Server 5.6, and MariaDB 10.

.. note:: As long as the master is using row based replication, this optimization is available on a TokuDB slave. This means that it's available even if the master is using InnoDB or MyISAM tables, or running non-TokuDB binaries.

Transactions and ACID-compliant Recovery
----------------------------------------

By default, TokuDB checkpoints all open tables regularly and logs all changes between checkpoints, so that after a power failure or system crash, TokuDB will restore all tables into their fully ACID-compliant state. That is, all committed transactions will be reflected in the tables, and any transaction not committed at the time of failure will be rolled back.

The default checkpoint period is every 60 seconds, and this specifies the time from the beginning of one checkpoint to the beginning of the next. If a checkpoint requires more than the defined checkpoint period to complete, the next checkpoint begins immediately. It is also related to the frequency with which log files are trimmed, as described below. The user can induce a checkpoint at any time by issuing the ``flush logs`` command. When a database is shut down normally it is also checkpointed and all open transactions are aborted. The logs are trimmed at startup.

Managing Log Size
-----------------

TokuDB keeps log files back to the most recent checkpoint. Whenever a log file reaches 100 MB, a new log file is started. Whenever there is a checkpoint, all log files older than the checkpoint are discarded. If the checkpoint period is set to be a very large number, logs will get trimmed less frequently. This value is set to 60 seconds by default.

TokuDB also keeps rollback logs for each open transaction. The size of each log is proportional to the amount of work done by its transaction and is stored compressed on disk. Rollback logs are trimmed when the associated transaction completes.

Recovery
--------

Recovery is fully automatic with TokuDB. TokuDB uses both the log files and rollback logs to recover from a crash. The time to recover from a crash is proportional to the combined size of the log files and uncompressed size of rollback logs. Thus, if there were no long-standing transactions open at the time of the the most recent checkpoint, recovery will take less than a minute.

Disabling the Write Cache
-------------------------

When using any transaction-safe database, it is essential that you understand the write-caching characteristics of your hardware. TokuDB provides transaction safe (ACID compliant) data storage for MySQL. However, if the underlying operating system or hardware does not actually write data to disk when it says it did, the system can corrupt your database when the machine crashes. For example, TokuDB can not guarantee proper recovery if it is mounted on an NFS volume. It is always safe to disable the write cache, but you may be giving up some performance.

For most configurations you must disable the write cache on your disk drives. On ATA/SATA drives, the following command should disable the write cache:

.. code-block:: bash

 $ hdparm -W0 /dev/hda

There are some cases when you can keep the write cache, for example:

* Write caching can remain enabled when using XFS, but only if XFS reports that disk write barriers work. If you see one of the following messages in /var/log/messages, then you must disable the write cache:

  * ``Disabling barriers, not supported with external log device``

  * ``Disabling barriers, not supported by the underlying device``

  * ``Disabling barriers, trial barrier write failed``

  XFS write barriers appear to succeed for single disks (with no LVM), or for very recent kernels (such as that provided by Fedora 12). For more information, see the `XFS FAQ <http://xfs.org/index.php/XFS_FAQ#Q:_How_can_I_tell_if_I_have_the_disk_write_cache_enabled.3F>`_.

In the following cases, you must disable the write cache:

* If you use the ext3 filesystem

* If you use LVM (although recent Linux kernels, such as Fedora 12, have fixed this problem)

* If you use Linux's software RAID

* If you use a RAID controller with battery-backed-up memory. This may seem counter-intuitive. For more information, see the `XFS FAQ <http://xfs.org/index.php/XFS_FAQ#Q:_How_can_I_tell_if_I_have_the_disk_write_cache_enabled.3F>`_

In summary, you should disable the write cache, unless you have a very specific reason not to do so.

Progress Tracking
-----------------

TokuDB has a system for tracking progress of long running statements, thereby removing the need to define triggers to track statement execution, as follows:

* Bulk Load: When loading large tables using ``LOAD DATA INFILE`` commands, doing a ``SHOW PROCESSLIST`` command in a separate client session shows progress. There are two progress stages. The first will state something like ``Inserted about 1000000 rows``. After all rows are processed like this, the next stage tracks progress by showing what fraction of the work is done (e.g. ``Loading of data about 45% done``)

* Adding Indexes: When adding indexes via ``ALTER TABLE`` or ``CREATE INDEX``, the command ``SHOW PROCESSLIST`` shows progress. When adding indexes via ``ALTER TABLE`` or ``CREATE INDEX``, the command ``SHOW PROCESSLIST`` will include an estimation of the number of rows processed. Use this information to verify progress is being made. Similar to bulk loading, the first stage shows how many rows have been processed, and the second stage shows progress with a fraction.

* Commits and Aborts: When committing or aborting a transaction, the command ``SHOW PROCESSLIST`` will include an estimate of the transactional operations processed.

Migrating to TokuDB
-------------------

To convert an existing table to use the TokuDB engine, run ``ALTER TABLE... ENGINE=TokuDB``. If you wish to load from a file, use ``LOAD DATA INFILE`` and not ``mysqldump``. Using ``mysqldump`` will be much slower. To create a file that can be loaded with ``LOAD DATA INFILE``, refer to the ``INTO OUTFILE`` option of the `SELECT Syntax <http://dev.mysql.com/doc/refman/5.5/en/select.html>`_.

.. note:: Creating this file does not save the schema of your table, so you may want to create a copy of that as well.

Hot Backup
----------

Hot Backup enables the database to be backed up with no downtime. The Tokutek hot backup library intercepts system calls that write files and duplicates the writes to the backup directory. An in depth design discussion of Hot Backup is available at:

* `TokuDB Hot Backup - Part 1 <http://www.tokutek.com/2013/09/tokudb-hot-backup-part-1/>`_

* `TokuDB Hot Backup - Part 2 <http://www.tokutek.com/2013/09/tokudb-hot-backup-part-1/>`_

Hot Backup 7.5.5 and later
**************************

.. contents::
   :local:

Configuration
^^^^^^^^^^^^^

Before using the Hot Backup plugin, the plugin must be installed. To install it, execute the following command::

 install plugin tokudb_backup soname 'tokudb_backup.so';

Once the plugin is installed, it is then possible to execute a backup. The destination directory where the backups will be located must be empty, otherwise a failure will occur. To back up a database, the user sets the ``tokudb_backup_dir`` variable to an empty directory as follows::

 set tokudb_backup_dir='/path_to_empty_directory';

As soon as the variable is set, the backup will begin.

Monitoring Progress and Checking Errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hot backup updates the *processlist* state while the backup is in progress. Users will be able to see the output by running ``show processlist`` or ``show full processlist``.

There are two variables that can be used to capture errors from Hot Backup. They are ``@@tokudb_backup_last_error`` and ``@@tokudb_backup_last_error_string``. When Hot Backup encounters an error, these will report on the error number and the error string respectively. For example, the following output shows these parameters following an attempted backup to a directory that was not empty:

.. code-block:: mysql
   :emphasize-lines: 1,3,9,11

 mysql> set tokudb_backup_dir='/tmp/backupdir';
 ERROR 1231 (42000): Variable 'tokudb_backup_dir' can't be set to the value of '/tmp/backupdir'
 mysql> select @@tokudb_backup_last_error;
 +----------------------------+
 | @@tokudb_backup_last_error |
 +----------------------------+
 |                         17 |
 +----------------------------+
 mysql> @@tokudb_backup_last_error_string;
 ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '@@tokudb_backup_last_error_string' at line 1
 mysql> select @@tokudb_backup_last_error_string;
 +---------------------------------------------------+
 | @@tokudb_backup_last_error_string                 |
 +---------------------------------------------------+
 | tokudb backup couldn't create needed directories. |
 +---------------------------------------------------+

Optional Settings
^^^^^^^^^^^^^^^^^

tokudb_backup_allowed_prefix
  This system-level variable restricts the location of the destination directory where the backups can be located. Attempts to backup to a location outside of the directory this variable points to or its children will result in an error. The default is null, backups have no restricted locations. This read-only variable can be set in the :file:`my.cnf` config file and displayed with the ``show variables`` command.

  .. code-block:: mysql
     :emphasize-lines: 1

   mysql> show variables where variable_name='tokudb_backup_allowed_prefix';
   +------------------------------+-----------+
   | Variable_name                | Value     |
   +------------------------------+-----------+
   | tokudb_backup_allowed_prefix | /dumpdir  |
   +------------------------------+-----------+

tokudb_backup_throttle
  This session-level variable throttles the write rate in bytes per second of the backup to prevent Hot Backup from crowding out other jobs in the system. The default and max value is 18446744073709551615.

  .. code-block:: mysql
     :emphasize-lines: 1

   mysql> set tokudb_backup_throttle=1000000;

tokudb_backup_dir
  When set, this session-level variable serves two purposes, to point to the destination directory where the backups will be dumped and to kick off the backup as soon as it's set.

tokudb_backup_last_error
  This session variable contains the error number from the last backup. 0 indicates success.

tokudb_backup_last_error_string
  This session variable contains the error string from the last backup.

Hot Backup Prior to 7.5.5
*************************

There is no requirement to install a plugin prior to 7.5.5. Hot Backup is compiled into the executable. To run Hot Backup, the destination directory must exist, be writable and empty. Once this directory is created, the backup can be run using the following command::

 backup to '/path_to_empty_directory';