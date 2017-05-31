.. Percona TokuDB documentation master file, created by
   sphinx-quickstart on Thu Jul  2 11:24:46 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. _dochome:

==================================
 Percona TokuDB - Documentation
==================================

.. warning:: 

  This documentation is considered legacy documentation and it has been deprecated once the TokuDB has been integrated into |Percona Server|. You can access the updated documentation in the |Percona Server| `5.6 <https://www.percona.com/doc/percona-server/5.6/tokudb/tokudb_intro.html>`_ and |Percona Server| `5.7 <https://www.percona.com/doc/percona-server/5.7/tokudb/tokudb_intro.html>`_. Most of the pages here will automatically redirect you to the newer version of the documentation. 

TokuDB® is a highly scalable, zero-maintenance downtime MySQL storage engine that delivers indexing-based query acceleration, improved replication performance, unparalleled compression, and live schema modification. TokuDB is a drop-in storage engine requiring no changes to MySQL or MariaDB applications or code and is fully ACID and MVCC compliant.

Additional features unique to TokuDB include:

* Up to 25x Data Compression

* Fast Inserts

* Eliminates Slave Lag with :ref:`Read Free Replication <read-free-repl>`

* Hot Schema Changes

  * Hot Index Creation - TokuDB tables support insertions, deletions and queries with no down time while indexes are being added to that table
  
  * Hot column addition, deletion, expansion, and rename - TokuDB tables support insertions, deletions and queries without down-time when an alter table adds, deletes, expands, or renames columns

* On-line Backup

For more information on installing and using TokuDB for MySQL and MariaDB, click on the following links:

.. toctree::
   :maxdepth: 1

   using-tokudb
   quickstart
   installation
   variables
   plugins
   ft-index
   faq
   troubleshooting
   appendix
   release-notes
   trademark-policy
   copyright

Getting the Most from TokuDB
----------------------------

**Compression**: TokuDB compresses all data on disk, including indexes. Compression lowers cost by reducing the amount of storage required and frees up disk space for additional indexes to achieve improved query performance. Depending on the compressibility of the data, we have seen compression ratios up to 25x for high compression. Compression can also lead to improved performance since less data needs to be read from and written to disk.

**Fast Insertions and Deletions**: TokuDB's Fractal Tree technology enables fast indexed insertions and deletions. Fractal Trees match B-trees in their indexing sweet spot (sequential data) and are up to two orders of magnitude faster for random data with high cardinality.

**Eliminates Slave Lag**: TokuDB replication slaves can be configured to process the replication stream with virtually no read IO. Uniqueness checking is performed on the TokuDB master and can be skipped on all TokuDB slaves. Also, row based replication ensures that all before and after row images are captured in the binary logs, so the TokuDB slaves can harness the power of Fractal Tree indexes and bypass traditional read-modify-write behavior. This "Read Free Replication" ensures that replication slaves do not fall behind the master and can be used for read scaling, backups, and disaster recovery, without sharding, expensive hardware, or limits on what can be replicated.

**Hot Index Creation**: TokuDB allows the addition of indexes to an existing table while inserts and queries are being performed on that table. This means that MySQL can be run continuously with no blocking of queries or insertions while indexes are added and eliminates the down-time that index changes would otherwise require.

**Hot Column Addition, Deletion, Expansion and Rename**: TokuDB allows the addition of new columns to an existing table, the deletion of existing columns from an existing table, the expansion of char, varchar, varbinary, and integer type columns in an existing table, and the renaming of an existing column while inserts and queries are being performed on that table.

**Online (Hot) Backup**: TokuDB can create backups of online database servers without downtime.

In practice, slow indexing often leads users to choose a smaller number of sub-optimal indexes in order to keep up with incoming data rates. These sub-optimal indexes result in disproportionately slower queries, since the difference in speed between a query with an index and the same query when no index is available can be many orders of magnitude. Thus, fast indexing means fast queries.

**Clustering Keys and Other Indexing Improvements**: TokuDB tables are clustered on the primary key. TokuDB also supports clustering secondary keys, providing better performance on a broader range of queries. A clustering key includes (or clusters) all of the columns in a table along with the key. As a result, one can efficiently retrieve any column when doing a range query on a clustering key. Also, with TokuDB, an auto-increment column can be used in any index and in any position within an index. Lastly, TokuDB indexes can include up to 32 columns.

**Less Aging/Fragmentation**: TokuDB can run much longer, likely indefinitely, without the need to perform the customary practice of dump/reload or ``OPTIMIZE TABLE`` to restore database performance. The key is the fundamental difference with which the Fractal Tree stores data on disk. Since, by default, the Fractal Tree will store data in 4MB chunks (pre-compression), as compared to InnoDB's 16KB, TokuDB has the ability to avoid "database disorder" up to 250x better than InnoDB.

**Bulk Loader**: TokuDB uses a parallel loader to create tables and offline indexes. This parallel loader will use multiple cores for fast offline table and index creation.

**Full-Featured Database**: TokuDB supports fully ACID-compliant transactions, MVCC (Multi-Version Concurrency Control), serialized isolation levels, row-level locking, and XA. TokuDB scales with high number of client connections, even for large tables.

**Lock Diagnostics**: TokuDB provides users with the tools to diagnose locking and deadlock issues. For more information, see :ref:`Lock Visualization in TokuDB <lock-visual>`.

**Progress Tracking**: Running ``SHOW PROCESSLIST`` when adding indexes provides status on how many rows have been processed. Running ``SHOW PROCESSLIST`` also shows progress on queries, as well as insertions, deletions and updates. This information is helpful for estimating how long operations will take to complete.

**Fast Recovery**: TokuDB supports very fast recovery, typically less than a minute.
