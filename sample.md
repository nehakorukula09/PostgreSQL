# Template for Restore cluster on a different host using pgbackrest

## Template Description: ##
This template outlines the steps to Restore cluster on a different host using pgbackrest in PostgreSQL on RHEL/CentOS and Ubuntu/Debian systems.

## Tags: ##
- {CLIENT_MONITOR_HOST} = The identifier of the client monitoring host from where we can connect to the databases
- {SSH_CONNECT_DB_HOST}= Host identifier for connecting to the database host via ssh_connect
- {RESTORING_PG_DB_HOST}= Host identifier for restoring DB Host or IP address.
- {BINDIR}: Binary directory location.
- {STANZA}: Name of the Stanza.

## Description of Work: ##

This process involves configuring New(Restoring) PostgreSQL host on RHEL/CentOS system and Ubuntu/Debian system to restore the cluster using pgbackrest.

## Pre-flight Checks: ##

- **On the Backup Node:**
  Backup Status
  ```bash
  pgbackrest --config=/etc/pgbackrest/pgbackrest.conf --stanza={STANZA} --log-level-console=detail info
  ```

- **On the new host (Restoring node):**
  1. PostgreSQL installed.
  2. pgbackest installed.
  3. passwordless SSH to and from DB node and backup node.

- **Same Version of PostgreSQL and pgbackrest:**
  Confirm that the DB servers are running the same version of PostgreSQL.
  ```bash
  {BINDIR}/postgres --version
  ```
  Confirm that pgbackrest version is same on DB nodes and backup node.
  ```bash
  pgbackrest version
  ```

- **Network Connectivity:**
  Ensure that the DB servers and backup servers are reachable over passwordless ssh vice-versa.

- **Disk Space:**
  Check for adequate disk space on DB server performing restore.
  ```bash
  df -khP
  ```


## Work Steps: ##

## Step 1: Connect to Monitor node: ##

- **1. Connect to Client monitor node.**
  ```
  ssh_ms connect {CLIENT_MONITOR_HOST}
  ```

## Step 2: Connect to Restoring DB node and validate data directory is empty. ##

- **1. Connect to the Restoring DB node.**
  ```
  ssh_connect {RESTORING_PG_DB_HOST}
  ```

- **2. Validate the cluster status, if Database is running.**
  Use below steps to validate the DB running status if it is running or proceed with the restore in an empty Data Directory.
  - **1. If RHEL/CentOS operating system.**
  ```bash
  psql -c "show data_directory;"
  ```
  Sample Output:
  ```bash
  data_directory     
  ------------------------
  /var/lib/pgsql/15/data
  (1 row)
  ```
  ```bash
  {BINDIR}/pg_ctl -D /var/lib/pgsql/15/data status
  ```
  Sample Output:
  ```bash
  pg_ctl: server is running (PID: 8593)
  /usr/pgsql-15/bin/postgres "-D" "/var/lib/pgsql/15/data/"
  ```
  - **2. If Ubuntu/Debian operating system.**
  ```bash
  pg_lsclusters 
  ```
  Sample Output:
  ```
  Ver Cluster Port Status Owner    Data directory              Log file
  14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
  ```
- **3. Stop the Database and Clear the contents of Data Directory.**


