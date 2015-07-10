.. _appendix:

========
Appendix
========

Fast Upserts and Updates
------------------------

Upserts (``INSERT ... ON DUPLICATE KEY UPDATE``) and Updates are slow because they are implemented as read row, modify row, write row. This implementation is needed because the number of rows affected by the update is returned, and the computation of affected rows requires one to know whether or not the row existed before the update or upsert operation. It also limits the throughput of these statements to read IO performance of the system.

On workloads where computing the number of affected rows is not important it is possible to execute these statements with no read requirement. The remainder of this section describes the conditions under which this optimization applies.

Example Tables
**************

Table with a simple primary key:

.. code-block:: sql

 CREATE TABLE table (
   id INT NOT NULL,
   column BIGINT NOT NULL,
   PRIMARY KEY(id)) ENGINE=TokuDB;

Table with a compound primary key:

.. code-block:: sql

 CREATE TABLE table (
   id_1 INT NOT NULL,
   id_2 BIGINT NOT NULL,
   column_int INT UNSIGNED NOT NULL,
   column_char CHAR(32),
   column_var VARCHAR(32),
   PRIMARY KEY(id_1,id_2)) ENGINE=TokuDB;

Currently Supported Update Expressions
**************************************

Set a char field to a constant string:

.. code-block:: sql

 column_char = 'Greetings!'

Set an integer field to a constant:

.. code-block:: sql

 column_int = 42

Add a constant to an integer field:

.. code-block:: sql

 column_int = column_int + 100

Subtract a constant from an integer field:

.. code-block:: sql

 column_int = column_int + 1

Decrement an integer field if its value is not already zero:

.. code-block:: sql

 column_int = if ( column_int = 0 , column_int - 1 )

More supported expressions will be added over time.

Example Upserts
***************

Insert a row. If the row already exists, then increment one of its columns.

.. code-block:: sql

 INSERT NOAR INTO table
   VALUES (1000,0)
   ON DUPLICATE KEY UPDATE column_int = column_int + 1;

Insert a row. If the row already exists, then increment one of its columns, and set another of its columns.

.. code-block:: sql

 INSERT NOAR INTO table
   VALUES (2000,3000,0,'Greetings!','Salutations!')
   ON DUPLICATE KEY UPDATE column_int = column_int + 1, 
   column_char = 'Greetings and Salutations!';

Requirements for Fast Upserts
*****************************

* The ``NOAR`` keyword is required. It stands for "no affected rows". Its presence forces the TokuDB storage engine to perform a fast operation or fail if it is unable to due to any of the following requirements not being met.

* The table must have a defined primary key.

* The table cannot have any secondary keys.

* No triggers on the table.

* If binary logging is enabled, then ``binlog_format`` must be set to ``STATEMENT``.

* The update fields must be int, char, or varchar types.

* Session must be in relaxed mode

  * ``SQL_MODE`` not set for ``STRICT_ALL_TABLES`` and ``STRICT_TRANS_TABLES``.

  * Numeric values are clipped at their maximum value.

Example Updates
***************

Set a column to a constant for the row with primary key 42. If the row does not exist, then nothing is changed.

.. code-block:: sql

 UPDATE NOAR table 
   SET column = 1
   WHERE id = 42;

Increment a column for the row with primary key 100. If the row does not exist, then nothing is changed.

.. code-block:: sql

 UPDATE NOAR table
   SET column = column + 1
   WHERE id = 100;

Decrement a column x and set column c to a constant string for the row with primary key ida 7 and idb 8. If the row does not exist, then nothing is changed.

.. code-block:: sql

 UPDATE NOAR table
   SET column_int = column_int - 1,
   column_char = 'Greetings!'
   WHERE id_1 = 7 AND id_2 = 8;

Requirements for Fast Updates
*****************************

* The ``NOAR`` keyword is required and stands for "no affected rows". Its presence forces the TokuDB storage engine to perform a fast operation or fail if it is unable to due to any of the following requirements not being met.

* The table must have a defined primary key.

* The primary key fields must have int, char or varchar type.

* Updated columns cannot be part of any secondary keys.

* The table cannot have any clustering keys.

* The ``WHERE`` condition must resolve to a single row.

* No triggers on the table.

* If binary logging is enabled, then ``binlog_format`` must be set to ``STATEMENT``.

* The update fields must be int, char, or varchar types.

* Session must be in relaxed mode

  * ``SQL_MODE`` not set for ``STRICT_ALL_TABLES`` and ``STRICT_TRANS_TABLES``.

  * Numeric values are clipped at their maximum value.

Compiling MySQL from Source
---------------------------

It is not necessary to build MySQL and the TokuDB for MySQL® handlerton from source, but if you want to this section tells you how to do so and how to link with the Tokutek Fractal Tree® Library.

Please note that Tokutek made some changes to the MySQL source that are required to either fix bugs or aid in performance, so if you are compiling from source you must use the Tokutek version of MySQL that is based on the MySQL 5.5.30 source.

The instructions in this section have been written with the assumption that you know what you are doing and are familiar with building the MySQL Community Server.

After executing these instructions, follow the instructions in the Tokutek Quick Start Guide to install and start the server.

System and Hardware Requirements
********************************

**Operating Systems**: These instructions were tested using CentOS 5.8. They are expected to work with other Linux distributions.

**Compiler**: These instructions were tested using gcc 4.1.2 and gcc-c++ 4.1.2

**Packages**: You will need the following packages installed at the noted revision or later:

* autoconf 2.61

* automake 1.10

* bison 2.3

* ccache 2.4

* cmake 2.6

* libtool 1.5

* ncurses-devel 5.6

* readline 5.2

* zlib-devel 1.2

**Processor Architecture**: TokuDB requires a 64-bit operating system

**Disk space**: 1 GB

Download and verify files
*************************

Download and verify the following files from the Tokutek web site and put them in the build directory:

* ``mysql-5.5.30-tokudb-7.1.0-0-src.tar.gz``
* ``mysql-5.5.30-tokudb-7.1.0-0-src.tar.gz.md5``
* ``tokufractaltreeindex-7.1.0-0-linux-x86 64.tar.gz``
* ``tokufractaltreeindex-7.1.0-0-linux-x86 64.tar.gz.md5``

Configure and build
*******************

.. note:: The C/C++ compilers default as gcc44 and g++44. You can override these by creating the environment variables CC and CXX or by modifying the tokudb.build.bash script directly on lines 16 and 17.

1. Extract the ``tokudb.build.bash`` script from the source tarball

 .. code-block:: console

  $ tar xzf mysql-5.5.30-tokudb-7.1.0-0-src.tar.gz

2. Move the script to the current directory:

 .. code-block:: console

  $ mv mysql-5.5.30-tokudb-7.1.0-0-src/scripts/tokudb.build.bash .

3. Make the script executable:

 .. code-block:: console

  $ chmod 744 tokudb.build.bash

4. Build the binary release tarball:

 .. code-block:: console

  ./tokudb.build.bash

Install
*******

Once the source has been compiled, you can install it by using the newly created tarball in the source directory and following the instructions in the Quick Start Guide on installing and starting the server.

3rd Party Libraries
-------------------

jemalloc
********

Copyright (C) 2002-2011 Jason Evans <jasone@canonware.com>.

All rights reserved.

Copyright (C) 2007-2010 Mozilla Foundation. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice(s), this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice(s), this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDER(S) "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER(S) BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Copyright (C) 2009-2011 Facebook, Inc.

All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

* Neither the name of Facebook, Inc. nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.