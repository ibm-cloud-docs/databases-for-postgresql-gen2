---

copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords: postgresql, databases, connection limits, terminating connections, postgresql connection pooling, postgres connection pooling, managing connections, gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Managing connections
{: #managing-connections}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

Connections to your {{site.data.keyword.databases-for-postgresql}} deployment use resources, so it is important to consider how many connections you need to tune your deployment's performance. PostgreSQL uses a `max_connections` setting to limit the number of connections (and resources that are consumed by connections) to prevent run-away connection behavior from overwhelming your deployment's resources.

You can check the value of `max_connections` with your [Manager user](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-manager-user) and [`psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

```sh
ibmclouddb=> SHOW max_connections;
 max_connections
-----------------
 115
(1 row)
```
{: .codeblock}

## Connection limits
{: #postgres-connection-limits}

At provision, {{site.data.keyword.databases-for-postgresql}} sets the maximum number of connections to your PostgreSQL database to **115**. 15 connections are reserved for the superuser to maintain the state and integrity of your database, and 100 connections are available for you and your applications. If the number of connections to the database exceeds the 100-connection limit, new connections fail and return an error.

```sh
FATAL: remaining connection slots are reserved for
non-replication superuser connections
```

Exceeding the connection limit for your deployment can cause your database to be unreachable by your applications.

You can check the number of connections to your deployment with the `Manager` user, `psql`, and `pg_stat_database`.

```sql
SELECT count(distinct(numbackends)) FROM pg_stat_database;
```
{: .codeblock}

If you need to figure out where the connections are going, you can break down the connections by database.

```sql
SELECT datname, numbackends FROM pg_stat_database;
```
{: .codeblock}

To further investigate connections to a specific database, query `pg_stat_activity`.

```sql
SELECT * FROM pg_stat_activity WHERE datname='ibmclouddb';
```
{: .codeblock}

## Terminating connections
{: #terminate-connections}

Your `Manager` user has the `pg_signal_backend` role. If you find connections that need to reset or be closed, the Manager user can use both [`pg_cancel_backend` and `pg_terminate_backend`](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SIGNAL-TABLE){: .external}. The `pid` of a process is found from the `pg_stat_activity` table.

- `pg_cancel_backend` cancels a connection's current query without terminating the connection, and without stopping any other queries that it might be running.

   ```sql
   SELECT pg_cancel_backend(pid);
   ```
   {: .codeblock}

- `pg_terminate_backend` stops the entire process and closes the connection.

   ```sql
   SELECT pg_terminate_backend(pid);
   ```
   {: .codeblock}

The Manager user does have the power to reset or close the connections for any user on the deployment except superusers. Be careful not to terminate replication connections from the `ibm-replication` user, as it interferes with the high-availability of your deployment.

### End connections
{: #end-connections}

Currently, ending (killing) connections is not supported via UI, CLI, or API on Gen 2. However, you can execute a `kill_all_connections()` functions from `psql` as `Manager` user.

## Connection pooling
{: #connection-pooling}

One way to prevent exceeding the connection limit and ensure that connections from your applications are being handled efficiently is through connection pooling. If you find yourself setting the {{site.data.keyword.databases-for-postgresql_full}} connection limit to more than 500 connections, you should seriously consider using connection pooling or reevaluating how to more efficiently use and maintain connections. Performance benchmarking in the PostgreSQL community suggests 500 connections or fewer to be optimal for database performance.

Many PostgreSQL driver libraries have connection pooling classes and functions. You need to consult your driver's documentation to implement connection pooling that is optimal for your use case. For example, the Python driver Psycopg2 has classes to handle connection pooling in your application. The Java PostgreSQL JDBC driver has methods for [connection pooling at both the application and application server level](https://jdbc.postgresql.org/documentation/datasource/){: .external}.

Alternatively, you can use a third-party tool such as [PgBouncer](https://pgbouncer.github.io/){: .external} to manage your application's connections.
