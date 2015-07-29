

=====================================
Differences Between TokuDB and InnoDB
=====================================

The main advantage of TokuDB over InnoDB is the use of Fractal Tree indexing, which increases I/O performance. TokuDB features unparalleled data compression, fast inserts, hot schema changes and backup.

The following table lists some technical differences between the two engines, which you should keep in mind when deciding to switch to TokuDB.

.. list-table::
   :header-rows: 1
   :stub-columns: 1
   
   * - 
     - InnoDB
     - TokuDB
   * - Maximum row size
     - 8 KiB
     - 32 MiB
   * - Foreign keys
     - Supported
     - Not supported
   * - Maximum number of indexes per table
     - 128
     - ???
   * - Maximum key prefix size
     - 767 bytes
     - ???
   * - Auto-increment column placement in keys
     - First column
     - Any column
   * - Locking
     - Is it different?
     - ???
   * - Full-text indexing
     - Supported (as of MySQL 5.6.4)
     - Not supported
   * - Geospatial indexes
     - Supported (as of MySQL 5.7.5)
     - Not supported
   * - Memcached interface to TokuDB tables
     - Not sure what this is about
     - ???

Should some of the following marketing differences also be added?

http://www.tokutek.com/tokudb-for-mysql/tokudb-vs-innodb/