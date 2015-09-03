

==================
Percona TokuBackup
==================

Percona TokuBackup is an open-source hot backup utility for MySQL servers running the TokuDB storage engine (including Percona Server and MariaDB). It does not lock your database during backup. The TokuBackup library intercepts system calls that write files and duplicates the writes to the backup directory.

.. contents::
   :local:

Installing From Binaries
------------------------

.. QUESTION: It will not be available as a plugin any more?

Compiling and Installing From Source Code
-----------------------------------------

.. TBD

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

.. TBD

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

