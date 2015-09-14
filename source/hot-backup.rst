

==================
Percona TokuBackup
==================

Percona TokuBackup is an open-source hot backup utility for MySQL servers running the TokuDB storage engine (including Percona Server and MariaDB). It does not lock your database during backup. The TokuBackup library intercepts system calls that write files and duplicates the writes to the backup directory.

.. contents::
   :local:

Installing From Binaries
------------------------

TokuBackup is included with Percona Server 5.6 and later versions. Installation can be performed with the ``ps_tokudb_admin`` script.

To install Percona TokuBackup:

1. Run ``ps_tokudb_admin --enable-backup`` to add the ``preload-hotbackup`` option into **[mysqld_safe]** section of :file:`my.cnf`.

  .. code-block:: console
    
    $ sudo ps_tokudb_admin --enable-backup
    Checking SELinux status...
    INFO: SELinux is disabled.

    Checking if preload-hotbackup option is already set in config file...
    INFO: Option preload-hotbackup is not set in the config file.

    Checking TokuBackup plugin status...
    INFO: TokuBackup plugin is not installed.

    Adding preload-hotbackup option into /etc/my.cnf
    INFO: Successfully added preload-hotbackup option into /etc/my.cnf
    PLEASE RESTART MYSQL SERVICE AND RUN THIS SCRIPT AGAIN TO FINISH INSTALLATION!

2. Restart mysql service

  .. code-block:: console

    $ sudo systemctl restart mysql

3. Run ``ps_tokudb_admin --enable-backup`` again to finish installation of TokuBackup plugin

  .. code-block:: console
    
    $ sudo ps_tokudb_admin --enable-backup                                     Checking SELinux status...
    INFO: SELinux is disabled.

    Checking if preload-hotbackup option is already set in config file...
    INFO: Option preload-hotbackup is set in the config file.

    Checking TokuBackup plugin status...
    INFO: TokuBackup plugin is not installed.

    Checking if Percona Server is running with libHotBackup.so preloaded...
    INFO: Percona Server is running with libHotBackup.so preloaded.

    Installing TokuBackup plugin...
    INFO: Successfully installed TokuBackup plugin.

Compiling and Installing From Source Code
-----------------------------------------

Since TokuBackup is part of Percona Server, refer to corresponding documentation for building Percona Server at `https://www.percona.com/doc/percona-server/5.6/installation.html`_

Making a Backup
---------------

To run Percona TokuBackup, the backup destination directory must exist, be writable and empty. Once this directory is created, the backup can be run using the following command:

.. code-block:: console

 mysql> backup to '/path_to_empty_directory';

You can change the backup directory by setting the ``tokudb_backup_dir`` variable to an empty directory as follows:

.. code-block:: console

 mysql> set tokudb_backup_dir='/path_to_empty_directory';

.. note:: Setting the ``tokudb_backup_dir`` variable automatically starts the backup process to the specified directory.

Restoring From Backup
---------------------

Percona TokuBackup does not have any functionality for restoring a backup. You can use :command:`rsync` or :command:`cp` to restore the files. You should check that the restored files have the correct ownership and permissions.

.. note:: Make sure that the datadir is empty and that MySQL server is shut down before restoring from backup. You can’t restore to a datadir of a running mysqld instance (except when importing a partial backup).

The following example shows how you might use the :commad:`rsync` command to restore the backup:

.. code-block:: console

  $ rsync -avrP /data/backup/ /var/lib/mysql/

Since attributes of files are preserved, in most cases you will need to change their ownership to *mysql* before starting the database server. Otherwise, the files will be owned by the user who created the backup.

.. code-block:: console

  $ chown -R mysql:mysql /var/lib/mysql

Advanced Configuration
----------------------

.. contents::
   :local:

Monitoring Progress
*******************

TokuBackup updates the *processlist* state while the backup is in progress. You can see the output by running ``show processlist`` or ``show full processlist``.

Excluding Source Files
**********************

You can exclude certain files and directories based on a regular expression set in the ``tokudb_backup_exclude`` session variable. If the source file name matches the excluded regular expression, then the source file is excluded from backup.

For example, to exclude all ``lost+found`` directories from backup, use the following command:

.. code-block:: 

 mysql> set tokudb_backup_exclude='/lost\\+found($|/)';

Throttling Backup Rate
**********************

You can throttle the backup rate using the ``tokudb_backup_throttle`` session-level variable. This variable throttles the write rate in bytes per second of the backup to prevent TokuBackup from crowding out other jobs in the system. The default and max value is 18446744073709551615.

.. code-block:: console

 mysql> set tokudb_backup_throttle=1000000;

Restricting Backup Target
*************************

You can restrict the location of the destination directory where the backups can be located using the ``tokudb_backup_allowed_prefix`` system-level variable. Attempts to backup to a location outside of the specified directory or its children will result in an error.

The default is null, backups have no restricted locations. This read-only variable can be set in the :file:`my.cnf` configuration file and displayed with the ``show variables`` command:

.. code-block:: console

 mysql> show variables where variable_name='tokudb_backup_allowed_prefix';
 +------------------------------+-----------+
 | Variable_name                | Value     |
 +------------------------------+-----------+
 | tokudb_backup_allowed_prefix | /dumpdir  |
 +------------------------------+-----------+


Reporting Errors
****************

Percona TokuBackup uses two variables to capture errors. They are ``@@tokudb_backup_last_error`` and ``@@tokudb_backup_last_error_string``. When TokuBackup encounters an error, these will report on the error number and the error string respectively. For example, the following output shows these parameters following an attempted backup to a directory that was not empty:

.. code-block:: console

 mysql> set tokudb_backup_dir='/tmp/backupdir';
 ERROR 1231 (42000): Variable 'tokudb_backup_dir' can't be set to the value of '/tmp/backupdir'

 mysql> select @@tokudb_backup_last_error;
 +----------------------------+
 | @@tokudb_backup_last_error |
 +----------------------------+
 |                         17 |
 +----------------------------+
 
 mysql> select @@tokudb_backup_last_error_string;
 +---------------------------------------------------+
 | @@tokudb_backup_last_error_string                 |
 +---------------------------------------------------+
 | tokudb backup couldn't create needed directories. |
 +---------------------------------------------------+

<<<<<<< HEAD
=======
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
>>>>>>> master
