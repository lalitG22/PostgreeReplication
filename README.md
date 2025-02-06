PostgreSQL Replication - The Ultimate Guide
Replication in PostgreSQL allows data to be copied from a primary server to one or more standby servers for high availability, backups, and performance optimization.

Types of Replication
Streaming Replication
Description: Full database copy using WAL logs.

Logical Replication
Description: Only selected tables are copied.

Method 1: Streaming Replication (Full Database Copy)
Step 1: Configure Primary Server (Sender)
Edit postgresql.conf

ini
wal_level = replica
max_wal_senders = 10
wal_keep_size = 256MB
hot_standby = on
synchronous_standby_names = '*'
archive_mode = on
archive_command = 'cp %p /archive/%f'
synchronous_commit = on
hot_standby_feedback = on
max_replication_slots = 10
wal_log_hints = on
wal_sender_timeout = 30s
wal_receiver_timeout = 30s
Edit pg_hba.conf (Access Rules)

ini
host    replication    replicator    0.0.0.0/0    md5     
Create Replication User

sql
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
Restart PostgreSQL

bash
systemctl restart postgresql
Step 2: Configure Standby Server (Receiver)
Copy data from the primary server:

bash
PGPASSWORD='<password>' pg_basebackup -h <Primary-IP> -U replicator -D /var/lib/postgresql/15/main -P -R
Restart PostgreSQL on both servers:

bash
systemctl restart postgresql
🎉 Streaming Replication is ready! 🚀

Method 2: Logical Replication (Selected Tables Only)
Step 1: Configure Primary Server (Sender)
Create Replication User

sql
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
Create Publication for Selected Tables

sql
CREATE PUBLICATION my_pub FOR TABLE users, orders;
Grant Permission to Replicator

sql
GRANT SELECT ON public.users TO replicator;
GRANT SELECT ON public.orders TO replicator;
Step 2: Configure Standby Server (Receiver)
Create Replication User

sql
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
Create Subscription to Receive Data

sql
CREATE SUBSCRIPTION my_sub
CONNECTION 'host=<Primary-IP> port=5432 dbname=mydb user=replicator password=yourpassword'
PUBLICATION my_pub;
Restart PostgreSQL on both servers:

bash
systemctl restart postgresql
🎉 Logical Replication is now active! 🚀

Troubleshooting & Tips
Issue	Solution
Cannot connect to primary server	Check pg_hba.conf & firewall rules.
Data not replicating	Ensure wal_level = replica and max_wal_senders is set correctly.
Logical replication not working	Verify publication and subscription names match.
Standby server lags behind	Increase wal_keep_size.
Standby queries are slow	Set hot_standby_feedback = on.
Final Summary
✅ Streaming Replication is for full database sync.

✅ Logical Replication is for selective table replication.

✅ Follow these steps carefully, and your PostgreSQL Replication will work smoothly! 🚀

Feel free to tweak this further if needed! Let me know if there's anything else I can assist you with.
