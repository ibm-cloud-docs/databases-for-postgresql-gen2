---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"


keywords: postgresql, databases, postgres logical replication, postgresql logical replication

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}


# Databases for PostgreSQL as a logical replication destination
{: #logical-replication}

{{site.data.keyword.databases-for-postgresql_full}} supports [logical replication](https://www.postgresql.org/docs/current/logical-replication.html){: .external}, where you can create a subcriber or a publisher. You can also set up your external PostgreSQL as a publisher and your {{site.data.keyword.databases-for-postgresql}} deployment as a subscriber, and replicate your data across from an external database into your deployment.

Logical replication is only available on deployments running PostgreSQL 10 or above. Links to the PostgreSQL documentation direct you to the current version of PostgreSQL. If you need documentation for a specific version, you can find links to different PostgreSQL versions on the PostgreSQL documentation page.
{: .tip}

## Configuring the publisher
{: #configure-publisher}

The external PostgreSQL instance is the publisher, and needs to be configured in order for your {{site.data.keyword.databases-for-postgresql}} deployment to connect and be able to pull the data in correctly.

### Publisher functions
{: #publisher-functions}

**`create_publisher`**

In each database you can create multiple publications to publish the changes into a different subcriber.

```sh
Arguments:

    publisher_name               The Unique name of publisher.
    for table                    To publish single/list of tables
    for all table                To publish all tables, along with future tables.
    for all tables in schema     To publish all tables in schema, along wth future tables.

Usage:
    exampledb=> CREATE PUBLICATION my_publication;
    CREATE PUBLICATION
```
{: pre}

**`list_publisher`**

 List the number of publications running in each database.

 ```sh
 Usage:
    exampledb=# select * from pg_publication;
    oid  |    pubname     | pubowner | puballtables | pubinsert | pubupdate | pubdelete | pubtruncate
    -------+----------------+----------+--------------+-----------+-----------+-----------+-------------
    16401 | my_publication |       10 | t            | t         | t         | t         | t
```
{: pre}

**`alter_publisher`**

Modify the definition of the publication.

```sh
Syntax:
ALTER PUBLICATION name ADD publication_object [, ...]
ALTER PUBLICATION name SET publication_object [, ...]
ALTER PUBLICATION name DROP publication_object [, ...]
ALTER PUBLICATION name SET ( publication_parameter [= value] [, ... ] )
ALTER PUBLICATION name OWNER TO { new_owner | CURRENT_ROLE | CURRENT_USER | SESSION_USER }
ALTER PUBLICATION name RENAME TO new_name


 Usage:
    exampledb=# ALTER PUBLICATION my_publication RENAME TO my_publication_new;
    ALTER PUBLICATION
```
{: pre}

**`drop_publisher`**

Drop/Remove the publication not in use.

```sh
Usage:
    exampledb=# drop publication my_publication_new;
    DROP PUBLICATION
```
{: pre}

Consider the following as prerequisites:

1. Configure the extenal (publisher) PostgreSQL [`wal_level=logical`](https://www.postgresql.org/docs/current/logical-replication-config.html#LOGICAL-REPLICATION-CONFIG-PUBLISHER){: .external}.
2. Every table that is selected for replication needs to contain a [Primary key](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-PRIMARY-KEYS){: .external}, or have [`REPLICA IDENTITY`](https://www.postgresql.org/docs/current/sql-altertable.html#replica-identity){: .external} set.
3. Your external (publisher) PostgreSQL needs to have a replication user that has the [PostgreSQL privilege `REPLICATION`](https://www.postgresql.org/docs/current/sql-createrole.html#replication){: .external}, and that user needs to have SELECT and USAGE privileges on the databases along with REPLICATION privileges.
4. The PostgreSQL you are replicating from needs to have [TLS/SSL enabled](https://www.postgresql.org/docs/current/ssl-tcp.html){: .external}.
5. Provide [BYPASSRLS](https://www.postgresql.org/docs/current/logical-replication-security.html) privileges to external (publisher) PostgreSQL replication user, if row-level-security is enabled.  

There are inherent limitations and some restrictions to logical replication outlined in the [PostgreSQL documentation](https://www.postgresql.org/docs/current/logical-replication-restrictions.html){: .external}. Review these before deciding that logical replication is appropriate for your use case.
{: .tip}

## Configuring the subscriber
{: #configure-subscriber}

To configure your {{site.data.keyword.databases-for-postgresql}} deployment and to make sure that your data is replicated across correctly, make sure of the following.

1. You need to create a database in your deployment with same name as the database you intend to replicate.
2. Logical Replication works at the table level, so every table you select to publish, you need to create in the subscriber before starting the logical replication process. (You can use [`pg_dump`](https://www.postgresql.org/docs/current/app-pgdump.html){: .external} to help.) The table on the subscriber does not need to be identical to its publisher counterpart. However, the table on the subscriber must contain at least every column present in the table on the publisher. Additional columns present in the subscriber must not have NOT NULL or other constraints. If they do, replication fails.

Native PostgreSQL subscription commands require superuser privileges, which are not available on {{site.data.keyword.databases-for-postgresql}} deployments. Instead, your deployment includes a set of functions that can be used to set and manage logical replication for the subscription.
{: .note}

Only the admin user that is provided by {{site.data.keyword.databases-for-postgresql}} has permissions to run the following replication commands that allow you to subscribe and replicate content from an external PostgreSQL publisher.
{: .tip}

### Subscriber functions
{: #subscriber-functions}

**`create_subscription`**

```sh
Arguments:
    subscription_name   Unique name to create the subscription channel with
    host_ip             Publisher hostname or public IP address
    port                Port number publisher is running on
    username            `admin` user created on the publisher
    password            Password of the `admin` user on the publisher
    db_name             The name of the database to be replicated
    publisher_name      The name of publisher channel on the publisher

    These additional configuration options are only supported in PostgreSQL 17 and later.
    copy_data           The copy_data parameter (true or false, default: true) determines whether the initial data from the publication should be copied to the subscriber when the subscription is created.
    failover            The failover parameter (true or false, default: false) enables support for automatic failover of the subscription between replicated nodes, ensuring continuity during primary node transitions.
    origin              The origin parameter (ANY or NONE, default: NONE) allows filtering of changes based on their origin. It helps exclude or include changes coming from specific nodes in multi-node or cascading replication setups.

Usage:
    exampledb=> SELECT create_subscription('subs1','130.215.223.184','5432','password','admin','exampledb','my_publication');

    PostgreSQL 17 and later
    exampledb=> SELECT create_subscription('subs1','130.215.223.184','5432','password','admin','exampledb','my_publication'  'true',  'ANY' , 'true');
```

**`delete_subscription`**

```sh
Arguments:
    subscription_name   Name the subscription channel to delete
    db_name             The name of the replicated database

Usage:
exampledb=> SELECT delete_subscription('subs1', 'exampledb');
```

**`list_subscriptions`**

```sh
Arguments:
    None

Usage:
    exampledb=> SELECT * FROM list_subscriptions();
```

**`disable_subscription`**

```bash
Arguments:
    subscription_name   Name the subscription channel to disable
    db_name             The name of the replicated database

Usage:
    exampledb=> SELECT disable_subscription('subs1','exampledb');
```

**`enable_subscription`**

```bash
Arguments:
    subscription_name   Name the subscription channel to enable
    db_name             The name of the replicated database

Usage:
    exampledb=> SELECT enable_subscription('subs1','exampledb');
```

**`subscription_slot_none`**

This function is used to set the slot name of a subscription to NONE. This is needed in order to delete a subscription so that a remote replication slot cannot be dropped or does not exist or never existed.

```bash
Arguments:
    subscription_name   Name the subscription channel to alter
    db_name             The name of the replicated database

Usage:
    exampledb=> SELECT subscription_slot_none('subs1','exampledb');
```

**`refresh_subscription`**  

This function is used to refresh a subscription on the subscriber after changes are made on the publisher, like adding or removing a table.

```sh
Arguments:
    subscription_name   Name the subscription channel to refresh
    db_name             The name of the replicated database

Usage:
    exampledb=> SELECT refresh_subscription('subs1','exampledb');
```
{: .codeblock}

## Setting up logical replication on the publisher
{: #setting-up-logical-replication-publisher}

To configure your external PostgreSQL as a publisher, perform the following steps.

1. Edit your local `pg_hba.conf` and add the following.

    ```text
    hostssl    replication            replicator         0.0.0.0/0      md5
    hostssl    all                    replicator         0.0.0.0/0      md5
    ```

    The "replicator" field is the user that you set up with the [PostgreSQL privilege `REPLICATION`](https://www.postgresql.org/docs/current/sql-createrole.html#replication){: .external}.

2. Edit your local `postgresql.conf` with the required [logical replication configuration](https://www.postgresql.org/docs/current/logical-replication-config.html){: .external}. Set `wal_level` to 'logical', and set `listen_addresses='*'` to accept connections from any host.  

    ```text
    listen_addresses='*'
    wal_level = logical
    ```
    {: .codeblock}

3. Restart your PostgreSQL server.

Now you can define a publisher on the database and add the tables that you want to replicate to the subscriber.

1. Log in to database you want to publish from with your replication user.

    ```sh
    psql -U replicator -d exampledb
    ```
    {: pre}

2. Create the publication channel.

    ```sh
    exampledb=> CREATE PUBLICATION my_publication;
    ```
    {: pre}

3. Add tables to publisher.

    ```sh
    exampledb=> ALTER PUBLICATION my_publication ADD TABLE my_table;
    ```
    {: pre}

    The number of workers that back the synchronization that is defined by the `max_logical_replication_workers` configuration parameter is limited and cannot be changed. Therefore, use the least possible number of publications and add as many tables as possible to one publication.

## Setting up logical replication on the subscriber
{: #setting-up-logical-replication-subscriber}

To configure your {{site.data.keyword.databases-for-postgresql}} deployment as a subscriber, perform the following steps.

1. Log in to the database created for replication with `admin` user.

    ```sh
    psql -U admin -d exampledb
    ```
    {: pre}

2. Run the following query to call the `create_subscription` function and create the subscriber channel.

    ```sh
    exampledb=> SELECT create_subscription('subs1','130.215.223.184','5432','admin','password','exampledb','my_publication');
    ```
    {: pre}

## Monitoring replication
{: #monitor-replication}

You can monitor the status of logical replication from both the publisher and the subscriber by running the following query on either.

```sh
exampledb=> SELECT * FROM pg_stat_replication;
```
{: pre}
