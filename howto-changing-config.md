---
copyright:
  years: 2019, 2024
lastupdated: "2024-09-25"

keywords: postgresql, databases, config, postgresql uri, postgresql logging integration, changing postgresql configuration, postgresql time zone, postgresql logging, postgresql connection uri, changing config, changing configuration

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Changing your {{site.data.keyword.databases-for-postgresql}} configuration
{: #changing-configuration}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

{{site.data.keyword.databases-for-postgresql_full}} allows you to change some of the PostgreSQL configuration settings so you can tune your PostgreSQL databases to your use case. To make permanent changes to the database configuration, use the {{site.data.keyword.databases-for}} [CLI plug-in](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-configuration) or [API](/apidocs/cloud-databases-api/cloud-databases-api-v5#updatedatabaseconfiguration){: external} to write the changes to the configuration file for your deployment.

The configuration is defined in a schema. To make a change, you send a JSON object with the settings and their new values to the API or the CLI. For example, to set the `max_connections` setting to 150, you would supply:

```sh
{"configuration":{"max_connections":150}}
```
{: .codeblock}

to the CLI or to the API. 

For more information, see [Managing PostgreSQL connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections). 

## Using the CLI with {{site.data.keyword.databases-for-postgresql}}
{: #using-cli}
{: cli}

You can check the default configuration of your deployment with the `deployment-configuration-schema` command.

```sh
ibmcloud cdb deployment-configuration-schema <INSTANCE_NAME_OR_CRN>
```
{: pre}

Similarly, change your configuration with the `deployment-configuration` command.

```sh
ibmcloud cdb deployment-configuration <INSTANCE_NAME_OR_CRN> [@JSON_FILE | JSON_STRING]
```
{: pre}

The command reads the changes that you would like to make from the JSON object or a file. For more information, see the [reference page](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-configuration).

## Using the API with {{site.data.keyword.databases-for-postgresql_full}}
{: #using-api}
{: api}

The two deployment-configuration endpoints allow viewing the configuration schema and changing the configuration. To view the configuration schema, send a `GET` request to `/deployments/{id}/configuration/schema`.

To change the configuration, send the settings that you would like to change as a JSON object in the request body of a `PATCH` request to `/deployments/{id}/configuration`.

For more information, see the [API reference](https://cloud.ibm.com/apidocs/cloud-databases-api#change-your-database-configuration){: external}.


## Available {{site.data.keyword.databases-for-postgresql_full}} configuration settings
{: #available-config-settings}

### {{site.data.keyword.databases-for-postgresql_full}} time zone settings
{: #mem-settings}

The time zone for {{site.data.keyword.databases-for-postgresql_full}} deployments is always UTC (Coordinated Universal Time). This setting is not configurable by clients.

### {{site.data.keyword.databases-for-postgresql_full}} memory settings
{: #mem-settings}

[`shared_buffers`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS){: .external}

- Default - `32000` (number of 8 KiB buffers, or about 262 MB) 
- Recommended max value: 25% of available RAM
- Restarts database? - **Yes** 
  
The recommended memory allocation for `shared_buffers` is 25% of the deployment's RAM. Setting `shared_buffers` any higher can result in memory issues that cause the database to crash, and might decrease the performance of your database as data is most likely buffered by the OS already. Setting `shared_buffers` to equal, close to equal, or higher than the amount of allocated memory prevents the database from starting. The setting specifies the number of 8 KiB shared memory buffers.
{: important}
  
For example, 1 GB of `shared_buffers` space is `1048576 KiB`, and (`1048576 KiB / 8 KiB`) is `131072` buffers. Your deployment can use extra RAM for caching and performance, even without allocating it to `shared_buffers`. You do not have to configure the database to use all of the allocated RAM in order for your deployment to use it.

For existing workloads, or when scaling RAM, increasing memory to shared buffers might not be the best course of action. Instead, track your table and index cache hit ratios. If the cache hit ratios are in the high nineties, let the OS use the memory in other areas instead of increasing `shared_buffers`.

You can use these queries as the `admin` user, or any user with the `pg_monitor` role to track the cache hit ratios:

#### Tables
{: #tables}

```sh
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit)  as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as table_hit_ratio
FROM 
  pg_statio_user_tables;
```
{: .pre}

#### Indexes
{: #indexes}

```sh
SELECT 
  sum(idx_blks_read) as idx_read,
  sum(idx_blks_hit)  as idx_hit,
  (sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit) as index_hit_ratio
FROM 
pg_statio_user_indexes;
```
{: .pre}

The `work_mem` value is automatically adjusted in relationship to the `shared_buffer` and `max_connection` configuration values.
{: note}

### {{site.data.keyword.databases-for-postgresql_full}} general settings

{: #gen-settings}

[`max_connections`](https://www.postgresql.org/docs/current/runtime-config-connection.html#GUC-MAX-CONNECTIONS){: .external}

- Default - 115
- Restarts database? - **YES**
- Notes - [You might need to scale before you increase max connections.](https://cloud.ibm.com/docs/databases-for-postgresql?topic=databases-for-postgresql-postgresql-ha-dr#connection-limits-ha)

[`max_locks_per_transaction`](https://www.postgresql.org/docs/current/runtime-config-locks.html#GUC-MAX-LOCKS-PER-TRANSACTION){: external}

- Default - 64
- Options - Minimum value of 10
- Restarts database? - **YES**

[`max_prepared_transactions`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-MAX-PREPARED-TRANSACTIONS){: .external}

- Default - `0`
- Restarts database? - **YES**
- Notes - The default value of `0` disables use of [prepared transactions](https://www.postgresql.org/docs/current/sql-prepare-transaction.html){: .external} and is recommended unless you need to use them.

[`synchronous_commit`](https://www.postgresql.org/docs/current/wal-async-commit.html){: .external}

- Default - `local`
- Restarts database - No
- Options - `local`, `on`, or `off`
- Notes - Setting `synchronous_commit` to off increases transaction commit rate at the expense of a loss of committed transactions if an unclean shutdown occurs. With `synchronous_commit` set to `on`, a transaction is committed only when written to the leader and at least one replica. Therefore, the `on` setting is only available on formations that have been horizontally scaled to at least three members. Before implementing this change, see [High availability](/docs/databases-for-postgresql?topic=databases-for-postgresql-postgresql-ha-dr).

[`effective_io_concurrency`](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-EFFECTIVE-IO-CONCURRENCY){: .external}

- Default - `12`
- Restarts database - No
- Notes - It is recommended to leave this setting at the default. Only increase it if you profiled SQL queries and have observed inefficient bitmap heap scans. As [IOPS is tied to disk size](/docs/databases-for-postgresql?topic=databases-for-postgresql-performance#disk-iops), increasing this setting on default or smaller sized disks is also not recommended.

[`deadlock_timeout`](https://www.postgresql.org/docs/current/runtime-config-locks.html){: .external}

- Default - `10000`
- Restarts database - No
- Options - Minimum value of 100
- Notes - The number of milliseconds to wait before checking for deadlock and the duration where lock waits are logged. Logs available through the [logging integration](/docs/cloud-databases?topic=cloud-databases-logging){: external}. Setting this number too low negatively impacts performance.

[`log_connections`](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-CONNECTIONS){: .external}

- Default - `off`
- Restarts database - No
- Options - Values of `on` or `off` 
- Notes - Setting this value to `on` makes the logs verbose. It also shows the connections of the monitoring tool as it extracts metrics every 60 seconds. When this is set to `on`, it is recommended to set the application_name in the connection URI to keep an overview in the logs, as the IP addresses shown are the Kubernetes internal IPs. Details about adjusting the connection URI are found in the [PostgreSQL documentation](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING){: .external}. When set to `off`, there is no change in behavior to the default setting and no connections are logged. Logs are available through the [logging integration](/docs/cloud-databases?topic=cloud-databases-logging). If `on` is set, the logs show lines similar to this example, where the application name is set as `test-app`:

```sh
2021-03-01 10:27:56 UTC [[unknown]] [00000] [708]: [2-1] user=admin,db=ibmclouddb,client=127.0.0.1 LOG:  connection authorized: user=admin database=ibmclouddb application_name=test-app SSL enabled (protocol=TLSv1.3, cipher=TLS_AES_256_GCM_SHA384, bits=256, compression=off)
```

[`log_disconnections`](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-DISCONNECTIONS){: .external}

- Default - `off`
- Restarts database - No
- Options - Values of `on` or `off` 
- Notes - Setting this value to `on` makes the logs verbose. It will also show the disconnections of the monitoring tools as it extracts metrics every 60 seconds. When this is set to `on`, it is recommended to set the application_name in the connection URI to keep an overview in the logs, as the IP addresses shown are the Kubernetes internal IPs. Details about adjusting the connection URI are found in the [PostgreSQL documentation](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING){: .external}. When set to `off`, there is no change in behavior to the default setting and no disconnections are logged. Logs are available through the [logging integration](/docs/cloud-databases?topic=cloud-databases-logging). If `on` is set, the logs show lines similar to this example where the application name is set as `test-app`:

```sh
2021-03-01 10:27:56 UTC [test-app] [00000] [708]: [3-1] user=admin,db=ibmclouddb,client=127.0.0.1 LOG:  disconnection: session time: 0:00:00.793 user=admin database=ibmclouddb host=127.0.0.1 port=50638
```    

[`log_min_duration_statement`](https://www.postgresql.org/docs/current/runtime-config-logging.html){: .external}

- Default - `100`
- Restarts database - No
- Options - Minimum value of 100
- Notes - Statements that take longer than the specified number of milliseconds are logged.  

[`tcp_keepalives_idle`](https://www.postgresql.org/docs/10/runtime-config-connection.html){: .external}

- Default - `111`
- Restarts database - No

[`tcp_keepalives_interval`](https://www.postgresql.org/docs/10/runtime-config-connection.html){: .external}

- Default - `15`
- Restarts database - No 

[`tcp_keepalives_count`](https://www.postgresql.org/docs/10/runtime-config-connection.html){: .external}

- Default - `6`
- Restarts database - No

### {{site.data.keyword.databases-for-postgresql_full}} WAL settings
{: #wal-settings}

[`archive_timeout`](https://www.postgresql.org/docs/current/runtime-config-wal.html){: .external}

- Default - `1800`
- Restarts database - No
- Options - Minimum value of 300
- Notes - The number of seconds to wait before forcing a switch to the next WAL file. If the number of seconds has passed and if there has been database activity, the server switches to a new segment. Effectively limits the amount of time data can remain unarchived.

The next three settings `wal_level`, `max_replication_slots` and `max_wal_senders` enable use of the [`wal2json` logical decoding plug-in](/docs/databases-for-postgresql?topic=databases-for-postgresql-wal2json). If you aren't using this plug-in, leave these settings at the default.

[`wal_level`](https://www.postgresql.org/docs/current/runtime-config-wal.html){: .external}

- Default - `replica`
- Restarts database - **YES**
- Notes - Controls WAL level. Allowed values are `replica` or `logical`. Set to `logical` to use logical decoding. If you are not using logical decoding, using `logical` increases the WAL size, which has several disadvantages and no real advantage.

[`max_replication_slots`](https://www.postgresql.org/docs/current/runtime-config-replication.html){: .external}

- Default - `10`
- Restarts database - **YES**
- Notes - The maximum number of simultaneously defined replication slots. The default and minimum number of slots is 10. Twenty slots are reserved for internal use by your deployment for High-Availability (HA) purposes. To use slots, you need to set the value above 20 and have one slot per consumer. Add one extra slot over the minimum per expected consumer. Using `wal2json` and not increasing `max_replication_slots` can impact HA and read-only replicas. If you are not using `wal2json`, leave this setting at the default.

[`max_wal_senders`](https://www.postgresql.org/docs/current/runtime-config-replication.html){: .external}

- Default - `12`
- Restarts database - **YES**
- Notes - The maximum number of simultaneously running WAL sender processes. The default, and minimum, is 12. One `wal_sender` per consumer is required. Twenty slots are reserved for internal use by your deployment for High-Availability (HA) purposes. You need to set the value above 20 and it is recommended to add one more `wal_sender` over the minimum per expected consumer. Using `wal2json` and not increasing `max_wal_senders` can impact HA and read-only replicas. If you are not using `wal2json`, leave this setting at the default.
