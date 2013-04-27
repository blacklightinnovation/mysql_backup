mysql_backup
============

Simple MySQL Backup Script



I have always wanted to have a quick utility to backup my MySQL databases without needing to install a bunch of software or type out a bunch of commands. In sheer lack of a solution, I have written a light weight scrip to achieve this very goal. In this post, I will explain the script in detail and provide the code so that you too can utilize its ease. Soon we will be publishing access to our SVN repository so that you can also contribute to the script as it progresses.
Prerequisites:

Linux/Unix OS
MySQL Client
MySQL Dump utility
Root Level Privileges(System and MySQL)

# Installing the MySQL Client

CentOS/RHEL

    yum install mysql-client

Ubuntu/Debian

    apt-get install mysql-client

# Setting up the script:

There are a few modifications that must be made in order to run the script against your MySQL databases. First you must specify your username, password, host, and port of your databases so that the script knows where to point to make a connection. If your host is not within your network then you will have to have Internet connectivity and access from the remote hosts IP in order to be able to connect to the remote server.
Replace the following fields with your information:

    #DB Host
    dbhost="127.0.0.1"
    #DB User
    dbuser="root-level-user"
    #DB Pass
    dbpass="somepasswd"
    #DB Port
    dbport="3306"

# Add a cron job for the script
    0 0 * * * root /path/to/script/mysql_backup >> /dev/null 2&>1

# How it works... We verify the Base Directory for the output of the script (ie. the backup files)

    # Base Directory for backups
    backup_dir="/mysql_backup/"

    #Verify the Directory Exists.. If not then create it
    if [ ! -d $backup_dir ];
    then
            `mkdir -p $backup_dir`
    fi

# Next we define our basic MySQL commands to be excuted

    # General DB Identification Commands
    dblist="show databases;"
    dbtables="show tables;"
    
    # MySQL Cient Commands
    mysqlcmd="mysql -u$dbuser -p$dbpass -h$dbhost -P $dbport -e "
    databases=`$mysqlcmd "$dblist"`

# Next we loop through the databases found and identify tables for each db and dump the tables to .sql files:

For each database found show tables and dump tables into /$backup_dir/$db/tablename.sql
    
    for db in $databases
    do
            # The MySQL Command will dump the header 'Databases'. 
            # If the DB name = 'Databases' then ignore and continue
            if [ ! "$db" = "Database" ];
            then
                    # Verify this database has a backup directory, if not then create it.
                    if [ ! -d "$backup_dir/$db" ];
                    then
                            `mkdir -p "$backup_dir/$db"`
                    fi
                    # Specify the database to backup
                    usedb="use $db;"
                    # Identify all tables in the given database
                    tables=`$mysqlcmd "$usedb $dbtables"`
                    # Loop through the tables of given DB and backup each individually
                    for tn in $tables
                    do
                            # Verify that the table is valid and not a header for the given DB
                            if [ ! "$tn" = "Tables_in_$db" ];
                            then
                                    # Execute the backup of the given table
                                    mysqldump="mysqldump -u$dbuser -p$dbpass -P $dbport $db $tn "
                                    `$mysqldump > "$backup_dir/$db/$tn.sql"`
                            fi
                    done
            fi
    done

Here is the output of the script. You can see that all of the databases found in this mysql instance have been backed up to thier own folder and the backup files are individual to each table.
OUTPUT:

    root@test-machine:~/# ls  /mysql_backup/
    information_schema  justin  justin_test  mysql
    root@test-machine:~/scripts# ls  /mysql_backup/information_schema/
    CHARACTER_SETS.sql                         COLUMNS.sql  GLOBAL_STATUS.sql     PLUGINS.sql                  ROUTINES.sql           SESSION_VARIABLES.sql  TABLES.sql
    COLLATION_CHARACTER_SET_APPLICABILITY.sql  ENGINES.sql  GLOBAL_VARIABLES.sql  PROCESSLIST.sql              SCHEMA_PRIVILEGES.sql  STATISTICS.sql         TRIGGERS.sql
    COLLATIONS.sql                             EVENTS.sql   KEY_COLUMN_USAGE.sql  PROFILING.sql                SCHEMATA.sql           TABLE_CONSTRAINTS.sql  USER_PRIVILEGES.sql
    COLUMN_PRIVILEGES.sql                      FILES.sql    PARTITIONS.sql        REFERENTIAL_CONSTRAINTS.sql  SESSION_STATUS.sql     TABLE_PRIVILEGES.sql   VIEWS.sql

    root@test-machine:~/# ls  /mysql_backup/mysql/
    columns_priv.sql               general_log.sql                help_topic.sql                 procs_priv.sql                 tables_priv.sql                time_zone_transition.sql
    db.sql                         help_category.sql              host.sql                       proc.sql                       time_zone_leap_second.sql      time_zone_transition_type.sql
    event.sql                      help_keyword.sql               ndb_binlog_index.sql           servers.sql                    time_zone_name.sql             user.sql
    func.sql                       help_relation.sql              plugin.sql                     slow_log.sql                   time_zone.sql                  
    root@test-machine:~/#

Make sure to add execute permissions on the script after downloading.

    #!/bin/bash
    ######################################################
    # MySQL Database Backup Utility
    # 
    # You must change the dbhost, dbuser, dbpass, and 
    # dbport parameters in oder to use this utility
    #
    ######################################################
    
    #DB Host
    dbhost="127.0.0.1"
    #DB User
    dbuser="root-level-user"
    #DB Pass
    dbpass="somepasswd"
    #DB Port
    dbport="3306"
    
    # Base Directory for backups
    backup_dir="/mysql_backup/"
    
    #Verify the Directory Exists.. If not then create it
    if [ ! -d $backup_dir ];
    then
      `mkdir -p $backup_dir`
    fi
    
    # General DB Identification Commands
    dblist="show databases;"
    dbtables="show tables;"
    
    # MySQL Cient Commands
    mysqlcmd="mysql -u$dbuser -p$dbpass -h$dbhost -P $dbport -e "
    databases=`$mysqlcmd "$dblist"`
    
    # For each database found show tables and dump tables into /$backup_dir/$db/tablename.sql
    for db in $databases
    do
    	# The MySQL Command will dump the header 'Databases'. If the DB name = 'Databases' then ignore and continue
    	if [ ! "$db" = "Database" ];
    	then
    		# Verify this database has a backup directory, if not then create it.
    		if [ ! -d "$backup_dir/$db" ];
    		then
    			`mkdir -p "$backup_dir/$db"`
    		fi
    
    		usedb="use $db;"
    		tables=`$mysqlcmd "$usedb $dbtables"`
    		for tn in $tables
    		do
    			if [ ! "$tn" = "Tables_in_$db" ];
    			then
    				mysqldump="mysqldump -u$dbuser -p$dbpass -P $dbport $db $tn "
    				`$mysqldump > "$backup_dir/$db/$tn.sql"`
    			fi
    		done
    	fi
    done


