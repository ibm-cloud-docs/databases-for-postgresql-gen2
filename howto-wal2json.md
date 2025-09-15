---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, wal2json

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Configuring `wal2json`
{: #wal2json}

{{site.data.keyword.databases-for-postgresql_full}} deployments support the [`wal2json`](https://github.com/eulerto/wal2json){: external} plug-in, enabling [logical decoding](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html){: external} on your deployment.

Note:

- **Deprecated:** This plug-in is deprecated in PostgreSQL versions 9.6 and 10.  
- **Supported:** Only available in PostgreSQL versions **11 and above**.

1. First, you need to [configure](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration) the `wal_level`, `max_replication_slots`, and `max_wal_senders` settings. Change the `wal_level` to `logical`. The `max_replication_slots`, and `max_wal_senders` both need to be set to a value greater than 20. {{site.data.keyword.databases-for-postgresql}} reserves 20 replication slots and WAL senders for current and future operational purposes.

   ```sh
   curl -X PATCH https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/configuration
     -H 'Authorization: Bearer <>'
     -H 'Content-Type: application/json'
     -d '{"configuration": {
           "wal_level": "logical",
           "max_replication_slots": 21,
           "max_wal_senders": 21
           }
         }'
   ```
   {: pre}

2. Set a password for the [`repl` user](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-repl-user).
   Any user's password can be changed by using the {{site.data.keyword.databases-for}} CLI plug-in [`cdb deployment-user-password`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-user-password) command or {{site.data.keyword.databases-for}} API [`/deployments/{id}/users/{username}`](https://cloud.ibm.com/apidocs/cloud-databases-api#set-database-level-user-s-password) endpoint. The `repl` user has REPLICATION privileges and the `wal2json` plug-in uses it after you set a password for it.

3. Create a replication slot on the database from the {{site.data.keyword.databases-for}} API. Send a POST request to the [`/deployments/{id}/postgresql/logical_replication_slots`](https://cloud.ibm.com/apidocs/cloud-databases-api#create-a-new-logical-replication-slot) endpoint.

   ```sh
   curl -X POST https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/postgresql/logical_replication_slots   -H 'Authorization: Bearer <>'
     -H 'Content-Type: application/json'
     -d '{"logical_replication_slot": {
          "name": "<slot_name>",
          "database_name": "<database_name>",
          "plugin_type": "wal2json"
          }
        }'
   ```
   {: pre}

   The plug-in type must be `wal2json`. The database must be an existing database. The slot name can contain only lowercase letters, numbers, and the underscore character. You can check the existence of the replication slot by connecting to any database and running the following command:

   ```sh
   SELECT * FROM pg_replication_slots WHERE slot_name = '<slot_name>';
   ```
   {: pre}

4. To test the plug-in, run `pg_recvlogical` from the command-line. The command is available with an installation of PostgreSQL. Use the host and port from your deployment, and the database and slot name you created through the API.

   ```sh
   PGSSLMODE=require pg_recvlogical -d <DATABASE NAME> -U repl -h <HOST> -p <PORT>    --slot <SLOT NAME> --start -o pretty-print=1 -f -
   ```
   {: pre}

5. Create a table on `ibmclouddb` and insert some data. Ensure that the inserts come out in the command-line that is running `pg_recvlogical`.

   Table creates do not appear.
   {: .note}

## `wal2json` considerations and tips
{: #wal2json-tips}

- Setting `wal_level` to `logical` increases the size of the WAL files because PostgreSQL needs more data to accomplish logical decoding. If you aren't using `wal2json`, leave `wal_level` at the default. Larger WAL files potentially mean that more disk space is required. Write throughput can decrease, along with replication lag that affects high availability and read-only replicas, and longer restore times from a backup.

- Logical decoding has a set of restrictions on what replicates. Some of these include schema/DDL, sequences, TRUNCATE, and Large Objects.

- When a controlled HA switchover happens, it is possible that replication events will be delivered more than once. Downstream applications must be able to handle events being delivered more than once.

- If you create a logical replication slot, and a consumer is not connected and consuming the changes, you run the risk of running your deployment out of disk space. The replication slot tells PostgreSQL to keep all the transaction logs that have the changes that the consumer needs. If nothing is consuming those changes, PostgreSQL continues collecting them until it is out of disk space. You can monitor disk space with the [{{site.data.keyword.monitoringfull}} integration](/docs/databases-for-postgresql?topic=databases-for-postgresql-monitoring). If you run out of space, you can [scale up disk](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling), which allows the database to start. Then, you can either start consuming the changes or drop the slot.

- You can check how much disk space is being used by a specific replication slot and whether that replication slot has an active consumer. Use the `admin` user to run one of the following commands:  

   **PostgreSQL 10.x and newer**

    ```sh
    SELECT slot_name, pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(),restart_lsn)) AS lag, active from pg_replication_slots WHERE slot_type='logical';
    ```
    {: pre}

   **PostgreSQL 9.x**

    ```sh
    SELECT slot_name, pg_size_pretty(pg_xlog_location_diff(pg_current_xlog_location(),restart_lsn)) AS lag, active FROM pg_replication_slots WHERE slot_type='logical';
    ```
    {: pre}

If you see higher than expected disk usage on your deployment, troubleshoot by checking that your replication slot has a consumer and isn't running your deployment out of disk space.  
