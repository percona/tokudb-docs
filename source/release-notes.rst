.. _release-notes:

=============
Release Notes
=============

TokuDB 7.x
----------

Version 7.5.5
*************

New Features

* Rebase onto MySQL 5.5.41
* Rebase onto MariaDB 5.5.41
* Added the ability to throttle optimize table
* Added the ability to optimize individual indexes of a table
* Added the ability to optimize a percentage of a fractal tree
* Enhanced lock timeout debugging by logging both the blocked and blocking sql
* Make alter table reorganize partition interruptable
* Hot Backup is now implemented as a plugin
* Add debug info to records_in_range to help debug query plan problems
* Added a variable to disable the read only slave check

Bug Fixes

* Fix a MySQL bug when updating a partitioned table

Upgrade Considerations

* The hot backup syntax has changed.
* Upgrade after a dirty shutdown of TokuDB 7.1.5+ is supported. Upgrading from any version prior to 7.1.5 requires a clean shutdown

Version 7.5.3
*************

New Features

* Rebase onto MySQL 5.5.40
* Rebase onto MariaDB 5.5.40

Bug Fixes

* Fix crash in TC_LOG_MMAP::get_active_from_pool
* Fix TokuDB info schema error handling in MariaDB 10
* Fix cardinality data for partitioned TokuDB tables
* Fix alter table analyze partition for TokuDB tables
* Fix issue where add/drop index may break TokuDB cardinality data

Upgrade Considerations

* TokuDB 7.5.3 uses file format version 27

* Upgrade after a dirty shutdown of TokuDB 7.1.5+ is supported. Upgrading from any version prior to 7.1.5 requires a clean shutdown

Version 7.5.2
*************

Bug Fixes

* Removed overlocking caused by before/after insert/update/delete triggers on primary key ranges
* Fixed issue where unique key violations were falsely reported

Upgrade Considerations

* TokuDB 7.5.2 uses file format version 27
* Upgrade after a dirty shutdown of TokuDB 7.1.5+ is supported. Upgrading from any version prior to 7.1.5 requires a clean shutdown

Version 7.5.0
*************

New Features

* Created capability for TokuDB replication slave servers to avoid IO for uniqueness checks and update/delete statements
* Added support for multiple source directories in hot backup (enterprise feature)
* Added bulk fetch algorithm to improve performance of select and delete statements
* Improved output of TokuDB information schema plugins, (now includes database, table, and index names separately)
* Rebase onto MySQL 5.5.39
* Rebase onto MariaDB 5.5.39
* Several Fractal Tree index improvements
* Fixed bug in XA recovery
* Added configuration parameter to avoid fsync() in XA prepare phase
* Upgraded jemalloc to version 3.6.0

Upgrade Considerations

* TokuDB 7.5.0 uses file format version 27
* Upgrade after a dirty shutdown of TokuDB 7.1.5+ is supported. Upgrading from any version prior to 7.1.5 requires a clean shutdown

Version 7.1.8
*************

TokuDB 7.1.8 was an unreleased version that contains several bug fixes.

Version 7.1.7
*************

Upgrade Considerations

* TokuDB 7.1.7 uses file format version 25.

* Upgrade after a dirty shutdown of TokuDB 7.1.5+ is supported. Upgrading from any version prior to 7.1.5 requires a clean shutdown.

MariaDB

* Rebase onto MariaDB 5.5.38

MySQL

* Rebase onto MySQL 5.5.38

MySQL and MariaDB

* Hot optimize for MySQL 5.6 and MariaDB 10.0 using alter recreate
* Use thd_ha_data and thd_ha_set_data functions
* Fail TokuDB initialization if jemalloc is not loaded
* Hot backup direct I/O fix
* Speed up unique key check after load
* Use row estimate to decide if bulk loader is used to insert rows
* TokuDB sets invalid proc info
* MDEV-6324 uninitialied var in tokudb discover3 function

Fractal Tree index changes

* Fix some TokuDB loader bugs when NPROC limit exceeded
* Fix the dbremove assert when NPROC limit exceeded
* Fix the thread pool assert when NPROC limit exceeded
* Fix a TokuDB loader bug that would unlink the in use fractal tree files
* Fix a file descriptor leak in the bulk loader

Version 7.1.6
*************

Upgrade Considerations

* TokuDB 7.1.6 uses file format version 25. Upgrade after a dirty shutdown of TokuDB 7.1.5 is supported. Upgrade after a clean shutdown of versions prior to TokuDB 7.1.5 is required.

* TokuDB 7.1.5 uses file format version 25. Upgrade after a clean shutdown of prior versions is required.

* TokuDB 7.1.0 through TokuDB 7.0.1 use file format version 24. Upgrade after a clean shutdown of versions prior to TokuDB 7.0 is required.

MariaDB

* Rebase onto MariaDB 5.5.37

MySQL

* Rebase onto MySQL 5.5.37

* RQG temporal replication caused crash in TokuDB

MySQL and MariaDB

* Replace into replication bug
* Unique secondary key overlocking
* MDEV-5646 analyze table may crash
* MDEV-5932 wrong result for create table select
* Open table stall due to lots of deletes in the tree
* SQL Bench test insert performance regression
* MDEV-4533 delete ignore and row replication
* i_s_tokudb_lock_waits_released test sometimes fails
* MDEV-5595 slow point query with deleted key
* MDEV-6162 Incorrect use of rw locks

Fractal Tree index changes

* Change table open cost from O(n) to O(log n)
* Bound point query Fractal Tree index search
* Fix impossibly long index creation time due to deleted rows at the left edge of the tree

Version 7.1.5
*************

Upgrade considerations

* Note that TokuDB v7.1.5 includes file format changes and will automatically upgrade prior versions of TokuDB. However, this is a one-way process and prior versions of TokuDB are not compatible with newer versions. If you require the ability to revert to an older version of TokuDB you must make a full backup of your data folder prior to running this new version.

MariaDB

* Rebase onto MariaDB 5.5.36

  * Fix MDEV-5458
  * Fix MDEV-5404
  * Fix MDEV-5396
  * Fix MDEV-5405

* Fix memory leak on view

* Fix handlersocket library dependency

MySQL

* Rebase onto MySQL 5.5.36
* Fix last lock timeout memory leak

MariaDB and MySQL

* Use Red Hat devtoolset-1.1 (with its gcc 4.7) to build mysql and mariadb1,2
* Blocked lock tree requests are not killable
* Get rid of metadata_db and tokudb_meta_mutex to remove causes of stalls during table open
* Update cardinality only on first table open to avoid possible lock tree race
* ``Field::null_ptr`` is not set properly, wrong result on a NOT IN subquery with NULL values, also known as MDEV-5399
* TokuDB crash after cancel of online create index, now it won't start
* Optimize temporary table crash,2
* Fix race in tokudb.i_s_tokudb_lock_waits_releases test,2
* HCAD update function does not initialize the entire update message
* uint3korr reads beyond the end of the buffer
* ``tokudb::analyze`` conditional jump uninitialized value
* tokudb leaks dbt arrays
* tokudb should destroy the tokudb_primary_key_bytes_inserted partitioned counter
* Cleanup use of thd_proc_info to not reference stack variables after function returns
* Alter table sometimes ignores a change in old and new null bytes which leads to broken row encoding
* Enum values not being actualized after alter table
* Alter table drop primary key fails,2,3
* Zero length character columns cause drop column to assert
* Uniqueness violation during alter table rebuild gets the -100010 error
* show create table does not always display the auto_increment value
* Slave exec mode idempotent is broken
* Fix ``tokudb::index_next_same`` to set table->status
* Alter table operations (only partition related?) corrupt nullable columns bitmap
* Cleanup mysql-test files with Windows style line endings
* Map tokudb_small to lzma and tokudb_fast to quicklz
* Use session variable to control lock timeout
* Use session variable to control loader memory size
* RQG tests percona_qa/percona_qa.yy and temporal/current_timestamp_6.yy cause a crash on cachetable,2

Several changes to Fractal Tree index, the highlights of which include:

* Run full garbage collection after injecting into an overfull leaf node
* Avoid stalling small transactions when lock escalation is running,2
* Mempool can grow uncontrollably if it is long lived
* Continuous checkpointing can starve a hot index operation

Version 7.1.0
*************

Features:

* Added ability for users to view lock information via information_schema.tokudb_trx, information_schema.tokudb_locks, and information_schema.tokudb_lock_waits_tables. More information is available in Appendix B.
* Changed default compression to zlib, tokudb_row_format=tokudb_zlib.
* Changed default basement node size to 64K, tokudb_read_block_size=65536.
* Changed default analyze time to 5 seconds, tokudb_analyze_time=5.
* Changed default loader behavior to compress intermediate files, tokudb_load_save_space=ON.
* Added server variable to control amount of memory allocated for each bulk loader, tokudb_loader_memory_size. In prior TokuDB versions each loader allocated 50% of the available TokuDB cache.
* Added session variable to control output of lock timeouts, tokudb_lock_timeout_debug.
* Added session variable to hold information relating to the last lock timeout for the current session, tokudb_last_lock_timeout.
* Added plug-in tokudb_trx.
* Added plug-in tokudb_locks.
* Added plug-in tokudb_lock_waits.
* Removed plug-in tokudb_user_data.
* Removed plug-in tokudb_user_data_exact.
* Changed table close behavior such that all data for the table remains in the cache (and is not flushed immediately)
* Changed checkpointing behavior such that the next checkpoint now begins tokudb_checkpointing_period seconds after the prior checkpoint started, not after it completed.

Bug fixes:

* Fixed bug where keys containing case-insensitive fields may not be abortable (Tokutek/ft-engine/issues/94).
* Fixed several stalls (Tokutek/ft-engine/issues/95, Tokutek/ft-engine/issues/73, Tokutek/ft- engine/issues/66).
* Fixed issue where converting InnoDB tables with key_block_size created TokuDB tables with small basement node sizing (Tokutek/ft-engine/issues/62).
* Backported MariaDB processlist "huge time" bug (Tokutek/mariadb/9).

Version 7.0.4
*************

Bug fixes:

* Disabled hot column expansion of text and blob data types. Expanding these types must be performed via a table rewrite by setting tokudb_disable_hot_alter=ON for the session.

Version 7.0.3
*************

Features:

* Improved performance of in-memory point queries and secondary index range queries (ft-index #5).
* Allow users to control fsync frequency via tokudb fsync log period server variable (#47).
* Server crash uses gdb (if available) to produce additional trace information for support (#49).

Bug fixes:

* "1034 Incorrect key file" issue (#52)
* Performance regression in sql-bench/wisconsin test (#51)
* Performance regression in iiBench (#48)
* Added plug-in to MTRv1 (#46).
* Return error if we try to create a table with a deprecated field type (#42).
* Server crash when open file limit reached (#30).
* Overactive assert in ha ``tokudb::external`` lock (#2).

Version 7.0.2
*************

Enterprise Edition released with support for online backup.

Version 7.0.1
*************

MySQL 5.5 and MariaDB 5.5 Improvements.

* Added support for Direct IO.
* Improved cardinality accounting for query planning.
* Improved row estimates for query planning.
* Read-only transaction performance improvements.

MariaDB 5.5 Improvements

* Added support for Extended Keys
* Added support for Index Condition Pushdown (ICP)
* Added support for Multi-Range Reads (MRR)

Fast-Updates Improvements

* Added NOAR syntax
* Added support for Statement Based Replication
* Added support for update of VARCHAR columns

Additional changes include:

* Modified cachetable eviction algorithm to favor ejection of large objects
* Added most of TokuDB engine status information to information schema.global status
* Improved shutdown time on systems with lots of tables
* Added detection of transparent huge pages (server shuts down)

TokuDB 6.x
----------

Version 6.6.7
*************

Bug fixes:

* Improved row estimates if within single basement node (#6285)
* Lock escalation when transactions relock (#6283)
* Splitting empty leaf nodes crashed server if also rebalancing (#6300)
* Race condition when multiple threads performing locks and unlocks on high concurrency workloads (#5979)

Version 6.6.5
*************

Bug fixes:

* INSERT ... SELECT statements were grabbing locks when isolation level ¡= READ COMMITTED (#5974)
* Lock escalation was not applied to transaction range buffers, causing large RSS (#5977)

Version 6.6.4
*************

Bug fixes:

* Fixed upgrade bug preventing multiple upgrades after v6.0 (#5902)

Version 6.6.3
*************

Changes include:

* Added optimization for certain UPDATE and INSERT ON DUPLICATE KEY UPDATE statements.
* Significant performance improvements for single threaded and concurrent workloads, both in- memory and larger than RAM.
* Auto-increment column now resets to default value on truncate table.
* MySQL 5.1 and MariaDB 5.2 are no longer supported.
* Bulk loader now optionally compresses intermediate files.
* Added new information_schema plugins tokudb_fractal_tree_block_map and tokudb_fractal_tree_info.
* Updated MariaDB 5.5 version to 5.5.28a
* Updated MySQL 5.5 version to 5.5.28

  * Added fix for security issue CVE-2012-5579 in MySQL 5.5.28 binaries (fix from MariaDB 5.5.28a).

Version 6.5.1
*************

Bug fixes:

* Hot Column Expand operation on a table with deleted rows could cause server crash (#5674)

* Simultaneously dropping an index and querying using the index could cause server crash (#5679)

Version 6.5.0
*************

Changes include:

* Hot column expansion for varchar, char, varbinary, and integer data types
* Support for hot operations on partitioned tables
* Lock and latch refinement in the cachetable
* QPS and concurrency improvements for reads on in-memory workloads

Version 6.1.1
*************

Bug fixes:

* Auto-upgraded tables (prior to v6.1.0) with significant insertions using auto-increment could cause server crash (#5435)
* HCR operations could starve on MySQL 5.5 and MariaDB 5.5 builds on busy servers (#5377)
* Dynamically changing row format on MariaDB 5.2 and MariaDB 5.5 builds could cause server crash (#5429)

Version 6.1.0
*************

Changes include:

* TokuDB is now the default storage engine
* sql_mode now defaults to NO_ENGINE_SUBSTITUTION, an error is returned if the requested storage engine is unavailable
* Added HCAD support to MySQL 5.5 version
* Updated MySQL 5.5 version to 5.5.24
* Added support for MariaDB 5.5 (5.5.25)
* Improved in-memory point query performance via lock/latch refinement
* Updated jemalloc to version 3
* Removed requirement that LD_PRELOAD and LD_LIBRARY_PATH be set prior to starting mysqld.
* Modifying table compression (tokudb_row_format) is now a hot operation
* Modifying auto_increment value is now a hot operation
* Added TokuDB_file_map plug-in to show relationship between indexes and files

Version 6.0.1
*************

Features:

* Auto-increment values can now be modified (the table is recreated via a slow alter operation)
* Compression type can now be modified (the table is recreated via a slow alter operation)

Bug fixes:

* Patched MySQL authentication bug into all builds (#5079)
* Fixed MySQL bug where slave could crash on XA transactions spanning multiple storage engines (#5001)
* Fixed server crash on the following conditions:

  * Insert ignore scenario into empty table (#5003)
  * Insert scenario into particular schema under MariaDB 5.2.10 (#5013)
  * Race condition in upgrade/checkpointing scenario (#5016)
  * Auto-upgrading MySQL 5.5 database with existing unapplied HCAD messages (#5023)

Version 6.0.0
*************

Changes include:

* Reduced checkpoint variability
* Performance enhancements for multi-client workloads
* Performance enhancements for in-memory workloads
* Added support for LZMA compression algorithm
* Added capability to define compression algorithm via DDL or the tokudb_row_format session variable
* Added ability to suppress client error messages in the server log via the tokudb_log_client_errors session variable
* Added support for XA
* Added compatibility with prior versions of TokuDB (4.2.x, 5.0.x, 5.2.x)
* Added support for MySQL 5.5
* Updated MySQL 5.1 version to 5.1.61
* Updated MariaDB 5.2 version to 5.2.10

TokuDB 5.x
----------

Version 5.2.7
*************

Changes include:

* Performance enhancement for multi-client workloads.
* Improved point-query performance.
* Ability to disable prefetching for range queries with LIMIT via the tokudb_disable_prefetching session variable.
* Reduced memory footprint.
* Improved CPU utilization on bulk loads.
* Improved range query performance.
* Increased maximum row size from 4MiB to 32MiB.
* Hot Column Rename is now supported.
* Hot Optimize Table is now supported. Note that Optimize Table does not rebuild indexes in TokuDB, since TokuDB indexes do not fragment. Instead, it flushes pending work induced by column additions and deletions.

Version 5.0.6
*************

Features:

* Added table create time and last update time to information_schema.tables and SHOW TABLE STATUS.

Bug fixes:

* Fixed a bug in 5.0.5 that could, under certain circumstances, cause a crash.

Version 5.0.5
*************

Features:

* SELECT FOR UPDATE is now supported.
* Changed to single file download.
* Created script to assist users building from source.
* Point update deadlocks are avoided since TokuDB uses write locks for these operations.
* Replace into deadlocks are avoided since TokuDB uses write locks for these operations.
* Added more information to engine status output.

Version 5.0.4
*************

Performance:

* Opening and closing very large tables is much faster than in 5.0.3.
* Adding blob and text columns is much faster than in 5.0.3.

Bug fixes:

* Fixed a bug in 5.0.3 that would cause a crash when creating a hot index that is a clustering key.
* Fixed a bug in 5.0.3 that could, under certain circumstances, cause a crash when adding a column. (For more information, see MySQL Bug 61017.)
* Fixed a bug in 5.0.3 that could, under certain circumstances, cause a crash when updating a table that has a blob.
* Fixed a bug in 5.0.3 that could, under rare circumstances, cause the creation of an incorrect index when a hot index was created simultaneously with a replace into operation.
* Fixed a bug in 5.0.3 that would cause SHOW TABLE STATUS to show an incorrect number of rows for tables with more than two billion rows.

Version 5.0.3
*************

Features:

* Hot Schema Changes:

  * Hot index creation is now supported. When an index is created, the database remains available (online) for all normal operations, including queries, inserts, deletes, and up- dates. Once the index creation completes, it is becomes immediately available for future queries. Any inserts/deletes/updates to the table during index creation are incorporated in the new index. This allows adding of indexes without having to suspend normal operation of other database applications.
  * Hot Column Addition/Deletion is now supported. After a brief interruption, the database remains available (online) for all normal operations, including queries, inserts, deletes, and updates during an alter table that adds or deletes columns. There is a small (seconds to a few minutes) time during which the table is locked because MySQL requires that tables be closed and reopened during an alter table.

* Snapshot isolation is now the default isolation level in TokuDB. TokuDB implements snapshot isolation with multi-version concurrency control (MVCC).
* Automatic upgrade of databases created by TokuDB versions 4.2 or later is now supported. It is not necessary to dump and load databases running under TokuDB 4.2 or later.
* Row locking is improved. Row locks are now limited solely by the memory allocated, not by their use per index and not by their total number.
* The tokudb_tmp_dir server variable is added.
* SHOW ENGINE STATUS is improved.
* Size of uncompressed user data is now available via the information schema. Commands

  * ``SHOW ENGINE TokuDB user_data;``

  * ``SHOW ENGINE TokuDB user_data_exact;``

  are now deprecated. The alternatives are, respectively:

  * ``SELECT * FROM information_schema.TokuDB_user_data;``

  * ``SELECT * FROM information_schema.TokuDB_user_data_exact;``

TokuDB 4.x
----------

Version 4.2.0
*************

Changes include:

* Fixed a bug in previous 4.x versions. The bug affects tables built using the bulk loader available in TokuDB versions 4.x which can cause inconsistencies in the tables and possibly the loss of some data. We recommend that all users use TokuDB version 4.2.0 or later to rebuild tables that were built using earlier versions of the bulk loader.

Version 4.1.1
*************

Bug fixes:

* Fixed a bug in 4.1.0.

Version 4.1.0
*************

Features:

* SAVEPOINT is now supported.
* Progress reporting of bulk loads is improved. Progress is now displayed as a percentage.

Performance:

* TokuDB 4.1.1 uses less virtual memory than TokuDB 4.0.0 when loading an empty table from a data file.
* TokuDB 4.1.1 loads an empty table from a data file faster than Tokudb 4.0.0.

Bug fixes:

* Fixed a bug where the server, while testing for clustering keys, would crash when running a join query.
* Fixed a bug that caused a crash in rare cases when simultaneously loading multiple empty tables from data files.
* Fixed a bug that caused a crash when multiple instances of TokuDB were started using a common directory.

Version 4.0.0
*************

Performance:

* Bulk Loading: TokuDB now optimizes for the case of creating a table or adding an index.
* Point queries: Point queries are now faster.
* INSERT IGNORE is now faster in many cases.

Logging Optimizations:

* The logs needed for ACID transactions are now considerably smaller. Furthermore, when a transaction commits, the space associated with logging for that transaction is reclaimed more quickly. Finally, smaller logs means faster recovery in many cases.

Features:

* READ COMMITTED isolation level.

TokuDB 3.x
----------

Version 3.1.0
*************

Features:

* Improved handling of a full disk: When disk free space is below a configurable reserve TokuDB disallows insertions so that more disk space can be made available or mysqld can be shut down cleanly without triggering a recovery on restart. Also, if the disk becomes completely full TokuDB will freeze, allowing the user to free some disk space and allowing the database to resume operation.

Performance:

* Faster group commits.
* Faster crash recovery.

Improved diagnostics:

* SHOW ENGINE STATUS has been improved to include information on free disk space and some new internal diagnostics.
* SHOW PROCESSLIST shows the progress of commits and aborts.

Bug fixes:

* Fixed bugs which sometimes caused recovery to require manual intervention. These include bugs related to truncated log files, e.g. when file system fills; recovery on empty log files; and recovery when there are files with names similar to log files. Recovery now works automatically, with no human intervention, in all these cases.
* Fixed a bug related to chars in non-default collations. Chars may now be used in non-default collations.
* Added session variable tokudb_pk_insert_mode to handle interaction of REPLACE INTO commands with triggers and row based replication.

Version 3.0.4
*************
Changes include:

* Upgraded from MySQL 5.1.36 to 5.1.43.
* This release supports two versions of MySQL, 5.1.43 and 5.5.1-m2.
* This release adds a new client session variable called tokudb_write_lock_wait so the user can control the amount of time a thread waits for write locks to be attained before timing out.
* This release adds support for group commit.

Version 3.0.3
*************

Changes include:

* This release supports two versions of MySQL, 5.1.36 and 5.5.1-m2.

Version 3.0.2
*************

Changes include:

* This release fixes an issue in the recovery code that may prevent full recovery following a crash. There is no work-around, and users testing crash recovery should upgrade immediately.

Version 3.0.1
*************

Changes include:

* This release fixes a bug, in which certain statements failed when run in the READ UNCOMMITTED isolation level. The statements include REPLACE INTO, INSERT IGNORE, and INSERT ... ON DUPLICATE KEY UPDATE.

Version 3.0.0
*************

Changes include:

* Full ACID-compliant transactions, including support for transactional statements such as BEGIN TRANSACTION, END TRANSACTION, COMMIT and ROLLBACK.
* New command SHOW ENGINE tokudb user_data returns the amount of data the user in the system.
* Bug fixes.

TokuDB 2.x
----------

Version 2.2.0
*************

Changes include:

* Increase in multi-threaded performance through the use of a fair scheduler for queries and inserts.
* Added support for command SHOW ENGINE tokudb STATUS.
* Provide more detailed progress indicators in SHOW PROCESSLIST.
* A script is provided to show how much tokudb user data is in the system. TokuDB pricing is based on this script.
* Faster bulk load into empty table in these scenarios:

  * ``LOAD DATA INFILE...``
  * ``INSERT INTO table SELECT *...``

Version 2.1.0
*************

Changes include:

* Support for InnoDB.
* Upgraded from MySQL 5.1.30 to 5.1.36.
* Faster indexing of sequential keys.
* Faster bulk loads on tables with auto-increment fields.
* Faster range queries in some circumstances.

Version 2.0.2
*************

Changes include:

* Blob support: the maximum row size has been increased from 80KiB to 4MiB.
* Performance Improvement: updates on tables with clustering indexes have become up to twice as fast.

Version 2.0.1
*************

Changes include:

* Crash safety for Windows.
* CREATE INDEX is now abort-able. An abort will typically take less than 2 minutes.
* Fixed a problem with REPLACE INTO or INSERT INTO ... ON DUPLICATE KEY on tables with blob/text fields that could cause a crash or an incorrect insertion under certain circumstances.

Version 2.0.0
*************

Changes include:

* Crash safety for Linux: TokuDB for MySQL now includes checkpointing so that after a power failure or a crash, each table comes up in a consistent state.