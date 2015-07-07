.. highlight:: mysql

.. _installation:

============
Installation
============

Tokutek provides binary versions of MySQL and MariaDB with a few minor changes to support TokuDB. The TokuDB, MyISAM and InnoDB storage engines are included in the distribution files.

The installation of MySQL from a tarball release file is described in the `MySQL 5.5 reference manual <http://dev.mysql.com/doc/refman/5.5/en/binary-installation.html>`_. The following instructions are based on it with information specific to TokuDB.

Downloading
-----------

To get started, download the appropriate binaries from the `TokuDB Download Page <http://www.tokutek.com/tokudb-for-mysql/download-community/>`_. The file name should follow this convention:

.. code-block:: none
   [mysql|mariadb]-[mysql|mariadb version]-tokudb-[tokudb version & build]-linux-x86_64.tar.gz.

For the purposes of this document, we'll use the name of the tarball corresponding to MySQL 5.5.30 with TokuDB v7.1.0. Make sure you use the name of the tarball corresponding to the file you downloaded.

After downloading, optionally verify the MD5 checksum by comparing the results of the following command to the MD5 checksum on the support page:

.. code-block:: bash
   $ md5sum mysql-5.5.30-tokudb-7.1.0-linux-x86_64.tar.gz

At this point, it is a good idea to determine the type of installation you wish to perform. The options are:

* :ref:`Installing TokuDB on a new server with no previous MySQL installation <new>`

* :ref:`Installing TokuDB to replace MySQL <replace>`

* :ref:`Upgrading from a previous TokuDB version <upgrade>`

* :ref:`Local or user space installation, optionally using MySQL Sandbox <local>`

.. _new:

Installing TokuDB from Scratch
------------------------------

All the operations have to be performed with root privileges because you will need to perform privileged operations. Please check with your specific Linux distribution the appropriate method to perform privileged operations.

1. Create a MySQL specific group. For CentOS distributions, the GID is set to 27. Other Linux distributions may vary, even across instances of the same distribution. It is a good idea to set a uniform GID across all servers. We will use GID 927 for the rest of this document.

 .. code-block:: bash
   $ groupadd -g 927 mysql

2. Create a MySQL system user. Similar to GID, the UID in CentOS distributions is set to 27. The user needs to be defined as a *system user*, which implies that it won't have a login shell and home directory. As with the GID, we will use UID 927 for the rest of this document.

 .. code-block:: bash
   $ useradd -r -u 927 -g mysql mysql

3. Create the installation base directory. You may specify a different sub-directory tree to follow your installations standard.

 .. code-block:: bash
   $ mkdir -pv /opt/tokutek

4. Extract the tarball you downloaded from our website under the base directory. You may use other directories (for example, :file:`/usr/local` or a private directory).

 .. code-block:: bash
   $ cd /opt/tokutek
   $ tar xvzf /path/to/mysql-5.5.30-tokudb-7.1.0-linux-x86_64.tar.gz

5. Create a symbolic link pointing to the newly created directory. This will simplify future upgrades and installations by redefining the symbolic link to a new directory.

 .. code-block:: bash
   $ ln -sv mysql-5.5.30-tokudb-7.1.0-linux-x86_64 mysql

6. Change the ownership of the new directory and it's contents to ``mysql:mysql``.

 .. code-block:: bash
   $ cd mysql
   $ chown -Rv mysql:mysql

7. Create a default configuration file. You will have to edit this file as needed. You may leave most of the default memory allocation values as-is to allow TokuDB to use as much RAM as possible.

 .. code-block:: bash
   $ cp -v support-files/my-small.cnf /etc/my.cnf

8. Add default data and binary directories, and mysqld user. Edit the :file:`/etc/my.cnf` file to specify the database directory, the location for the binaries and the user to use for the mysqld process. These values are going to be needed to make sure the next step succeeds. Add the following lines in the ``[mysqld]`` section of the file replacing the actual values as needed:

 .. code-block:: none
   datadir = /var/lib/mysql
   basedir = /opt/tokutek/mysql
   user = mysql

9. You might need to edit the resulting :file:`/etc/my.cnf` further to accommodate for other non-default settings. Create the system tables by running the following script. If needed, you may specify the options ``basedir`` and ``datadir`` with the proper paths before the ``--user`` option:

 .. code-block:: bash
   $ scripts/mysql_install_db --user=mysql

10. Create a link in :file:`/etc/init.d` to start TokuDB's server daemon as a service.

  First, edit the :file:`mysql.server` file and make sure that ``basedir`` is set to ``/opt/tokutek/mysql`` (or the directory under which you installed the tarball). This is not needed if the variables are set up in the :file:`/etc/my.cnf` file.

  Then, create a symbolic link under :file:`/etc/init.d` pointing to the :file:`mysql.server` script:

  .. code-block:: bash
     $ ln -sv /opt/tokutek/mysql/support-files/mysql.server /etc/init.d/mysql

11. Start MySQL service.

 .. code-block:: bash
   $ service mysql start

You may want to modify your ``PATH`` environment variable to include the sub-directory with the client utilities: :file:`/opt/tokutek/mysql/bin`.

Please refer to :ref:`Verify the Installation <verify>` to confirm that everything is running as expected.

.. _upgrade:

Upgrading From Previous Releases of TokuDB
------------------------------------------

TokuDB v7.1.0 supports automatic upgrades from TokuDB versions 4.1, 5.x, 6.x, and 7.x using the same configuration and data files from the older versions.

It is highly recommended that you have a full working backup of your data before upgrading any MySQL version. The backup should include the MySQL configuration files as well.

All the operations have to be performed with root privileges because you will need to perform privileged operations. Please check with your specific Linux distribution the appropriate method to perform privileged operations.

1. Create the installation base directory, unless it already exists. You may specify a different subdirectory tree to follow your installations standard.

 .. code-block:: bash
   $ mkdir -pv /opt/tokutek

2. Extract the tarball you downloaded from our website under the base directory.

 .. code-block:: bash
   cd /opt/tokutek
   tar xvzf /path/to/mysql-5.5.30-tokudb-7.1.0-linux-x86_64.tar.gz

3. Shutdown the existing instance. Make sure that it was a clean shutdown. It is important to make sure there are no pending transactions in the log files prior to the upgrade to avoid any potential data corruption.

 .. code-block:: bash
    $ mysqladmin shutdown

4. Create a symbolic link pointing to the newly created directory.

 .. code-block:: bash
    $ ln -sv mysql-5.5.30-tokudb-7.1.0-linux-x86_64 mysql

5. Create a link in :file:`/etc/init.d` to start TokuDB's server daemon as a service.

  First, edit the :file:`mysql.server` file and make sure that ``basedir`` is set to ``/opt/tokutek/mysql`` (or the directory under which you installed the tarball). This is not needed if the variables are set up in the :file:`/etc/my.cnf` file.

  Then, create a symbolic link under :file:`/etc/init.d` pointing to the :file:`mysql.server` script:

  .. code-block:: bash
     $ ln -sv /opt/tokutek/mysql/support-files/mysql.server /etc/init.d/mysql

6. Start the new MySQL instance. Make sure there were no errors after the startup completed by inspecting the error log.

 .. code-block:: bash
    $ service mysql start

7. Run :command:`mysql_upgrade` to update the system tables with any changes since the last release. It is important that you run the :command:`mysql_upgrade` utility provided with our binaries.

 .. code-block:: bash
    $ /opt/tokutek/mysql/bin/mysql_upgrade

8. Update the plug-ins. TokuDB v7.1 introduced 3 new plug-ins and removed 2 existing ones. Log in to MySQL and execute the following commands if upgrading a TokuDB database that was prior to v7.1 before the upgrade:

 .. code-block::
    INSTALL PLUGIN tokudb_trx SONAME 'ha_tokudb.so';
    INSTALL PLUGIN tokudb_locksSONAME 'ha_tokudb.so';
    INSTALL PLUGIN tokudb_lock_waitsSONAME 'ha_tokudb.so';
    DELETE FROM mysql.plugin WHERE NAME LIKE 'tokudb_user_data%';

Please refer to :ref:`Verify the Installation <verify>` to confirm that everything is running as expected. You should also verify the application data.

.. _replace:

Replace an Existing MySQL Instance
----------------------------------

The procedure to replace an existing MySQL instance should be the same as :ref:`Upgrading From Previous Releases of TokuDB <upgrade>`. After converting your tables to TokuDB, you should comment out the global options related to memory caches (``innodb_buffer_pool%``).

.. note:: When upgrading from MySQL 5.1 using InnoDB plug-in to MySQL 5.5 or MariaDB, you need to comment out the lines specifying the InnoDB plug-in libraries in your :file:`my.cnf` file. To verify whether or not the options are enabled, you can use the following command:
   .. code-block:: bash
      $ my_print_defaults mysqld | grep "ha_inno"

The previous command should return no results for TokuDB to start cleanly.

.. note:: As of TokuDB version 6.1.0, it is set as the default storage engine. When migrating from an existing MySQL or MariaDB installation, you need to specify the default storage engine to InnoDB or MyISAM before the first run. Set one of the following options in the ``[mysqld]`` section of the :file:`my.cnf` file:

  * ``default-storage-engine=MyISAM``
  * ``default-storage-engine=InnoDB``

This setting can be removed once the installation has been completed.

After running the ``mysql_upgrade`` script, run the following SQL commands to setup the TokuDB plug-in. Use the :command:`mysql` utility. You will need to login into MySQL with a user with sufficient privileges to run the ``install plugin`` command:

.. code-block::
   INSTALL PLUGIN TokuDB SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_file_map SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_fractal_tree_info SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_fractal_tree_block_map SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_trx SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_locks SONAME 'ha_tokudb.so';
   INSTALL PLUGIN tokudb_lock_waits SONAME 'ha_tokudb.so';
   SET GLOBAL default_storage_engine=TokuDB;

Please refer to :ref:`Verify the Installation <verify>` to confirm that everything is running as expected. You should also verify the application data.

.. _local:

Local or User Space Installation
--------------------------------

It is possible to run TokuDB as any regular user. To do so, you can extract the tarball in a private directory and run the commands from there. You have to make sure that a number of conditions are met to avoid having permission issues:

* No other MySQL instance should be running on the machine. If there is another one in place, you have to make sure your configuration will use a different port, log and data files, and configuration files to avoid conflicts.

* Make sure that you set up the log and data directories to a path where your user has full read and write access

* Read the MySQL Documentation section on running multiple instances for full details on how to specify the different options.

* The variable ``tokudb_cache_size`` is set to 50% of physical RAM on the server. When running multiple instances of MySQL, you may need to adjust it to a more sensible value to avoid memory over allocation, which may cause the server to become unresponsive or crash different processes, including MySQL instances.

If you decide to use this option, consider using MySQL Sandbox. MySQL Sandbox offers a number of tools to install one or more MySQL instances as a regular user taking care of all configuration details automatically. Each instance can be run independently or it can set up different replication topologies as well. As a result of the installation process, the end user will have a number of scripts to start, stop and connect to each individual instance without the need to be aware of the configuration and runtime details.

The procedure to install TokuDB for MySQL Sandbox is no different than regular MySQL. You can read the instructions to create a single sandbox here.

Installing TokuDB for MariaDB on Ubuntu Server
----------------------------------------------

TokuDB for MariaDB has an unmatched dependency on Ubuntu Server v12.04. Our MariaDB build is dependent on libevent v1.4, Ubuntu Servers install libevent v2.0 by default.

When trying to start the MySQL service without installing the proper library, it will fail with the following error in the log file:

.. code-block:: none
   /opt/tokutek/mysql/bin/mysqld:
   error while loading shared libraries: libevent-1.4.so.2:
   cannot open shared object file: No such file or directory

To verify which version you have in place you may use the following command:

.. code-block:: bash
   :emphasize-lines: 1

   $ dpkg-query -l libevent-*

   | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
   |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
   ||/ Name                        Version                      Description
   +++-===========================-============================-========================================
   ii  libevent-1.4-2              1.4.14b-stable-0ubuntu1      asynchronous event notification library
   ii  libevent-2.0-5              2.0.16-stable-1 Asynchronous event notification library

If the line for libevent-1.4 is not in the list, you may install it by issuing the following command:

.. code-block:: bash
   $ sudo apt-get install libevent-1.4

.. _verify:

Verify the Installation
-----------------------

Start a mysql client session to verify the TokuDB storage engine installation.

.. code-block:: bash
   $ /opt/tokutek/mysql/bin/mysql

The default socket for mysqld is :file:`/tmp/mysql.sock`. At the command prompt, execute the following command:

.. code-block::
   SHOW PLUGINS;

The output should include the following lines:

.. code-block:: none
   | TokuDB                        | ACTIVE | STORAGE ENGINE     | ha_tokudb.so ...
   | TokuDB_file_map               | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...
   | TokuDB_fractal_tree_info      | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...
   | TokuDB_fractal_tree_block_map | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...
   | TokuDB_trx                    | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...
   | TokuDB_locks                  | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...
   | TokuDB_lock_waits             | ACTIVE | INFORMATION SCHEMA | ha_tokudb.so ...

Also execute the following command:

.. code-block::
   SHOW ENGINES;

The output should include the following line:

.. code-block:: none
   | TokuDB | YES | Tokutek TokuDB Storage Engine