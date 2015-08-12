

==========
Hot Backup
==========

Hot Backup enables the database to be backed up with no downtime. The Hot Backup library intercepts system calls that write files and duplicates the writes to the backup directory.

An in depth design discussion of Hot Backup is available in the following blog posts:

* `TokuDB Hot Backup - Part 1 <http://www.tokutek.com/2013/09/tokudb-hot-backup-part-1/>`_

* `TokuDB Hot Backup - Part 2 <http://www.tokutek.com/2013/09/tokudb-hot-backup-part-2/>`_

As of TokuDB 7.5.5, the Hot Backup library is distributed as a plug-in. For more information about installing and using the Hot Backup plug-in with TokuDB 7.5.5 and later versions, see the following sections:

.. contents::
   :local:

.. note:: There is no requirement to install the Hot Backup plug-in for versions prior to 7.5.5, because Hot Backup is compiled into the executable. To run Hot Backup, the destination directory must exist, be writable and empty. Once this directory is created, the backup can be run using the following command:

 .. code-block:: console

  mysql> backup to '/path_to_empty_directory';

Configuration
-------------

Before using the Hot Backup plug-in, it must be installed. To install the plug-in, execute the following command:

.. code-block:: console

 mysql> install plugin tokudb_backup soname 'tokudb_backup.so';

Once the plug-in is installed, it is then possible to execute a backup. The destination directory where the backups will be located must be empty, otherwise a failure will occur. To back up a database, the user sets the ``tokudb_backup_dir`` variable to an empty directory as follows:

.. code-block:: console

 mysql> set tokudb_backup_dir='/path_to_empty_directory';

As soon as the variable is set, the backup will begin.

Monitoring Progress and Checking Errors
---------------------------------------

Hot Backup updates the *processlist* state while the backup is in progress. Users will be able to see the output by running ``show processlist`` or ``show full processlist``.

There are two variables that can be used to capture errors from Hot Backup. They are ``@@tokudb_backup_last_error`` and ``@@tokudb_backup_last_error_string``. When Hot Backup encounters an error, these will report on the error number and the error string respectively. For example, the following output shows these parameters following an attempted backup to a directory that was not empty:

.. code-block:: console

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
-----------------

``tokudb_backup_allowed_prefix``
  This system-level variable restricts the location of the destination directory where the backups can be located. Attempts to backup to a location outside of the directory this variable points to or its children will result in an error. The default is null, backups have no restricted locations. This read-only variable can be set in the :file:`my.cnf` config file and displayed with the ``show variables`` command.

  .. code-block:: console

   mysql> show variables where variable_name='tokudb_backup_allowed_prefix';
   +------------------------------+-----------+
   | Variable_name                | Value     |
   +------------------------------+-----------+
   | tokudb_backup_allowed_prefix | /dumpdir  |
   +------------------------------+-----------+

``tokudb_backup_dir``
  When set, this session-level variable serves two purposes, to point to the destination directory where the backups will be dumped and to kick off the backup as soon as it's set.

``tokudb_backup_exclude``
 Use this variable to set a regular expression that defines source files excluded from backup. For example, to exclude all `lost+found` directories, use the following command:

 .. code-block:: console

   mysql> set tokudb_backup_exclude='/lost\\+found($|/)';

``tokudb_backup_last_error``
  This session variable contains the error number from the last backup. 0 indicates success.

``tokudb_backup_last_error_string``
  This session variable contains the error string from the last backup.


``tokudb_backup_throttle``
  This session-level variable throttles the write rate in bytes per second of the backup to prevent Hot Backup from crowding out other jobs in the system. The default and max value is 18446744073709551615.

  .. code-block:: console

   mysql> set tokudb_backup_throttle=1000000;
