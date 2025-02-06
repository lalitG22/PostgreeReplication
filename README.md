# PostgreSQL Replication Guide

## Introduction

Replication in PostgreSQL allows data to be copied from one database server (Primary) to another (Standby). It helps in high availability, load balancing, and disaster recovery.

## Types of Replication

PostgreSQL supports different types of replication:

1. **Streaming Replication (Full Database Replication)**
   - Replicates the entire database.
   - Uses WAL (Write-Ahead Logging) for data synchronization.
   - Requires physical backups.
2. **Logical Replication (Table-Level Replication)**
   - Replicates specific tables.
   - Uses a publisher-subscriber model.
   - Allows selective data synchronization.

---

## Configuration Changes

To set up replication, changes are required in PostgreSQL configuration files:

### 1. **`postgresql.conf`**** (Primary Server)**

Modify the following properties:

- `wal_level = replica`

  - Determines how much information is written to WAL.
  - Values: `minimal`, `replica`, `logical`.
  - Use `replica` for streaming replication and `logical` for logical replication.

- `max_wal_senders = 10`

  - Defines the number of concurrent standby servers.
  - Increase this if you have multiple replicas.

- `wal_keep_size = 256MB`

  - Specifies how much WAL data to retain for standby servers.
  - Increase it if standby servers lag frequently.

- `hot_standby = on`

  - Enables read queries on the standby server.

- `synchronous_standby_names = '*'`

  - If using synchronous replication, define standby servers that must confirm transactions.

- `archive_mode = on` & `archive_command = 'cp %p /archive/%f'`

  - Used for archiving WAL logs.

- `primary_conninfo = 'host=<IP> user=replicator password=<password>'`

  - Required on standby server to connect to primary.

- `synchronous_commit = on`

  - Ensures transaction durability.

- `hot_standby_feedback = on`

  - Prevents long-running queries on standby from being terminated.

- `max_replication_slots = 5`

  - Defines the number of replication slots allowed.

- `wal_log_hints = on`

  - Required for `pg_rewind` tool.

- `wal_sender_timeout = 30s` & `wal_receiver_timeout = 30s`

  - Helps detect connection failures faster.

---

### 2. **`pg_hba.conf`**** (Primary Server)**

This file controls which IPs and users can connect to PostgreSQL.
Add:

```
host    replication    replicator    0.0.0.0/0    md5
```

or for a specific IP:

```
host    replication    replicator    <IP>/32    md5
```

After making changes, restart PostgreSQL:

```
systemctl restart postgresql
```

---

## Replication Methods

### **Method 1: Logical Replication (Table-Level)**

#### **Primary Server (Publisher) Configuration**

1. Create a replication user:
   ```sql
   CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
   ```
2. Create a publication:
   ```sql
   CREATE PUBLICATION my_pub FOR TABLE users, orders;
   ```
3. Grant access to the replication user:
   ```sql
   GRANT SELECT ON public.users TO replicator;
   ```

#### **Standby Server (Subscriber) Configuration**

1. Create a replication user:
   ```sql
   CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
   ```
2. Create a subscription:
   ```sql
   CREATE SUBSCRIPTION my_sub
   CONNECTION 'host=<PrimaryIP> port=5432 dbname=<DBName> user=replicator password=yourpassword'
   PUBLICATION my_pub;
   ```

---

### **Method 2: Streaming Replication (Full Database Replication)**

#### **Primary Server Configuration**

(Same configuration as mentioned earlier.)

#### **Standby Server Configuration**

Run the following command to take a backup from the primary server:

```sh
PGPASSWORD='yourpassword' pg_basebackup -h <PrimaryIP> -U replicator -D /var/lib/postgresql/15/main -P -R
```

**Explanation:**

1. `PGPASSWORD='yourpassword'` → Sets the password for authentication.
2. `pg_basebackup` → Copies primary server data.
3. `-h <PrimaryIP>` → Specifies the primary server's IP.
4. `-U replicator` → Uses the replication user.
5. `-D /var/lib/postgresql/15/main` → Destination directory for backup.
6. `-P` → Shows progress.
7. `-R` → Enables replication mode automatically.

After backup, restart PostgreSQL on standby:

```sh
systemctl restart postgresql
```

---

## Troubleshooting & Best Practices

- **Check Replication Status:**
  ```sql
  SELECT * FROM pg_stat_replication;
  ```
- **Ensure WAL logs are available:**
  ```sh
  ls -lh /var/lib/postgresql/15/main/pg_wal/
  ```
- **Check network connectivity:**
  ```sh
  ping <PrimaryIP>
  ```
- **Ensure firewall allows PostgreSQL port (default: 5432).**
  ```sh
  sudo ufw allow 5432/tcp
  ```
- **If using Docker, ensure container has correct networking.**

---

## Conclusion

PostgreSQL replication is essential for **high availability**, **load balancing**, and **disaster recovery**. Choosing between **Streaming Replication** and **Logical Replication** depends on your use case:

- Use **Streaming Replication** for full database synchronization.
- Use **Logical Replication** for selective table replication.

By following these steps, you can set up a **robust PostgreSQL replication environment** efficiently! 🚀

## Additional Monitoring Queries
View Replication Slots
To monitor the replication slots on your PostgreSQL server, use the following query:

 ```sh
SELECT * FROM pg_replication_slots;
  ```
This query will display all replication slots, which are used for tracking the replication state and ensuring data consistency between primary and standby servers.

View Active Replication Subscriptions
To check the status of active replication subscriptions, you can use:
 ```sh
SELECT * FROM pg_stat_subscription;
  ```
This query will show information about all active subscriptions, such as the replication state, the number of changes being replicated, and any errors related to the subscriptions.

With these additional monitoring queries, you can track the health and performance of your PostgreSQL replication setup effectively.

