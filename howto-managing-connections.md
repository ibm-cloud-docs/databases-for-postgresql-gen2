---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, connection limits, terminating connections, postgresql connection pooling, postgres connection pooling, managing connections

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Managing connections
{: #managing-connections}

Connections to your {{site.data.keyword.databases-for-postgresql}} deployment use resources, so it is important to consider how many connections you need to tune your deployment's performance. PostgreSQL uses a `max_connections` setting to limit the number of connections (and resources that are consumed by connections) to prevent run-away connection behavior from overwhelming your deployment's resources.

You can check the value of `max_connections` with your [admin user](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-admin-user) and [`psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

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

You can check the number of connections to your deployment with the admin user, `psql`, and `pg_stat_database`.

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

Your Admin user has the `pg_signal_backend` role. If you find connections that need to reset or be closed, the Admin user can use both [`pg_cancel_backend` and `pg_terminate_backend`](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SIGNAL-TABLE){: .external}. The `pid` of a process is found from the `pg_stat_activity` table.

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

The admin user does have the power to reset or close the connections for any user on the deployment except superusers. Be careful not to terminate replication connections from the `ibm-replication` user, as it interferes with the high-availability of your deployment.

### End connections
{: #end-connections}

If your deployment reaches the connection limit or you are having trouble connecting to your deployment and suspect that a high number of connections is a problem, disconnect all of the connections to your deployment.

In the UI, on the _Settings_ tab, there is a button to `End connections` to your deployment. Use caution, as it disrupts anything that is connected to your deployment.

The CLI command to end connections to the deployment is:

```sh
ibmcloud cdb deployment-kill-connections <DEPLOYMENT_NAME_OR_CRN>
```

You can also use the [{{site.data.keyword.databases-for}} API](/apidocs/cloud-databases-api/cloud-databases-api-v5#killconnections) to perform the end all connections operation.

## Connection pooling
{: #connection-pooling}

One way to prevent exceeding the connection limit and ensure that connections from your applications are being handled efficiently is through connection pooling. If you find yourself setting the {{site.data.keyword.databases-for-postgresql_full}} connection limit to more than 500 connections, you should seriously consider using connection pooling or reevaluating how to more efficiently use and maintain connections. Performance benchmarking in the PostgreSQL community suggests 500 connections or fewer to be optimal for database performance.

Many PostgreSQL driver libraries have connection pooling classes and functions. You need to consult your driver's documentation to implement connection pooling that is optimal for your use case. For example, the Python driver Psycopg2 has classes to handle connection pooling in your application. The Java PostgreSQL JDBC driver has methods for [connection pooling at both the application and application server level](https://jdbc.postgresql.org/documentation/datasource/){: .external}.

Alternatively, you can use a third-party tool such as [PgBouncer](https://pgbouncer.github.io/){: .external} to manage your application's connections.

## Raising the connection limit
{: #raise-connection-limit}

PostgreSQL allocates some amount of memory on a per-connection basis, typically around 5 - 10 MB per connection. It is important to consider the total amount of memory that is available to your deployment before increasing the connection limit. To raise the connection limit, first you might want to [scale your deployment](/docs/databases-for-postgresql?topic=databases-for-postgresql-resources-scaling) to ensure that you have enough memory to accommodate more connections.

Next, change the value of `max_connections` on your deployment. To make permanent changes to the [PostgreSQL configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration#changing-configuration), you want to use the {{site.data.keyword.databases-for}} [cli-plugin](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-configuration) or [API](/apidocs/cloud-databases-api/cloud-databases-api-v5#updatedatabaseconfiguration) to write the changes to the configuration file for your deployment.

For example, to raise `max_connections` to 215, it might be a good idea to scale your deployment to at least 2 GB of RAM per data member, for a total of 4 GB of RAM for your deployment. Once the scaling operation has finishes, then set the connection limit. 
1. Before you adjust `max_connections`, make sure to target your preferred region with a command like:


```sh
ibmcloud target -r <REGION>
```
{: pre}

2. Next, increase the amount of memory available to a deployment group with a command like:

 ```sh
ibmcloud cdb deployment-groups-set deployment-example member --memory 4096
```
{: pre}

3. Lastly, adjust `max_connections` with a command like:

```sh
ibmcloud cdb deployment-configuration <DEPLOYMENT_NAME_OR_CRN> '{"configuration":{"max_connections":215}}'
```
{: pre}

To make the changes through the API, use the following command:

```sh
curl -X PATCH `https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups/member' \
-H "Authorization: Bearer $APIKEY" \
-H "Content-Type: application/json" \
-d '{"memory": {
        "allocation_mb": 4096
      }
    }'

curl -X PATCH 'https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/configuration' \
-H "Authorization: Bearer $APIKEY" \
-H "Content-Type: application/json" \
-d '{"configuration":{
        "max_connections":215
      }
    }'
```
{: pre}

### Connection limits and TCP/IP keepalives settings
{: #keepalives}

In the event of a network connection or failover, it is possible that broken TCP/IP connections remain in a half-opened/closed state until the TCP keepalive timeouts are reached. To avoid this scenario, set the `socket_timeout` and `connection_timeout` settings in your specific application drivers, as well. The correct settings _vary based on the specific workload and it is important to run load tests before going to production_. A good starting point for the `connection_timeout` is 2 - 5 seconds. For the `socket_timeout`, a good starting point is 30 - 60 seconds.

Furthermore, on the server side, the following [keepalive configurations](https://www.postgresql.org/docs/12/runtime-config-connection.html){: .external} are used as the default.

- `tcp_keepalives_idle` is set to 5 minutes
- `tcp_keepalives_interval` probe interval is set to 10 seconds
- `tcp_keepalives_count` is set to 6

To prevent half-open/closed connections or bursts in connection attempts from overwhelming your deployment, set the [`max_connections` parameter](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration) for Postgres to at least double your expected connection count.

If your connection limit is reached, you can [end all connections](/docs/databases-for-postgresql?topic=databases-for-postgresql-managing-connections#end-connections) immediately.
