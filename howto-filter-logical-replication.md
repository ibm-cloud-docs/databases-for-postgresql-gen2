---

copyright:
  years: 2025
lastupdated: "2025-09-15"


keywords: postgresql, databases, postgres logical replication, postgresql logical replication, filter logical replication

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}


# Databases for PostgreSQL as a filter logical replication destination
{: #filter-logical-replication}

{{site.data.keyword.databases-for-postgresql_full}} supports [filter logical replication](https://www.postgresql.org/docs/15/logical-replication-row-filter.html){: .external}, where you can create  a publisher based on  **row-level filtering** and **column-list filtering** in logical replication, allowing fine-grained control over the data replicated to subscribers.

Filtered logical replication extends this capability by allowing you to specify which published tables and columns should be replicated to each subscriber. This filtering happens at the publication level and enables you to choose exactly what data needs to be replicated to each subscriber. This can be particularly useful in scenarios where not all data needs to be available on every node, for reasons related to security, performance, or data isolation.
{: .tip}

## Prerequisites
{: #filter-logical-replication-prereq}

- **PostgreSQL 15 or later** on publisher and subscriber.
- Logical replication must be enabled. For more information, see [logical replication](/docs/databases-for-postgresql?topic=databases-for-postgresql-logical-replication).
- Replication user must have appropriate privileges.

## Configuring row filters in logical replication
{: #configure-publisher-with-filter-rows}

A publisher with filtered replication for your {{site.data.keyword.databases-for-postgresql}} publishes only selected rows and/or columns of tables to subscribers using logical replication filters.

### Publisher functions
{: #filter-logical-replication-publisher-functions}

- **`create_publisher_with_filtered_rows`**

    In each database, you can create multiple publications with row and column filters to selectively replicate changes to different subscribers based on specific data requirements.

    ```sh
    Arguments:
    
        publisher_name               The Unique name of publisher.
        for table                    To publish single/list of tables
        for all table                To publish all tables, along with future tables.
        for all tables in schema     To publish all tables in schema, along wth future tables.
    
    Usage:
        exampledb=>CREATE PUBLICATION my_publication;
        CREATE PUBLICATION
        exampledb=>ALTER PUBLICATION my_publication ADD TABLE department_employee WHERE (department_id = 'development');
    
        //alternative
    
        exampledb=>CREATE PUBLICATION my_publication_development_hr FOR TABLE department_employee WHERE (department_id = 'development' or  department_id='hr');
        CREATE PUBLICATION
    
    ```
    {: pre}

- **`list_publisher`**

     List the number of publications running in each database, or you can see the table definition for active publications in a particular table.
    
     ```sh
     Usage:
        exampledb=>SELECT  * FROM pg_publication;
        oid  |            pubname            | pubowner | puballtables | pubinsert | pubupdate | pubdelete | pubtruncate | pubviaroot
        -------+-------------------------------+----------+--------------+-----------+-----------+-----------+-------------+------------
        16525 | my_publication                |    16421 | f            | t         | t         | t         | t           | f
        16527 | my_publication_development_hr |    16421 | f            | t         | t         | t         | t           | f
    ```
    {: pre}

- **`alter_publisher`**

    Modify the definition of the publication.
    
    ```sh
    Syntax:
    ALTER PUBLICATION name ADD publication_object [, ...]
    ALTER PUBLICATION name DROP publication_object [, ...]
    
     Usage:
        exampledb=>ALTER PUBLICATION my_publication  ADD TABLE department  WHERE (dept_name = 'hr');
        ALTER PUBLICATION
    ```
    {: pre}

- **`drop_publisher`**

    Drop/Remove table from the publication not in use.
    
    ```sh
    Usage:
        exampledb=>ALTER PUBLICATION my_publication drop table department;
        DROP PUBLICATION
    ```
    {: pre}


- **`create_publisher_with_filtered_columns`**

    Only replicates selected columns from a table.
    
    ```sh
    Usage:
        exampledb=>CREATE PUBLICATION employee_publication FOR TABLE employee (id, first_name, last_name);
        CREATE PUBLICATION
    
        // PostgreSQL >= 17, support combined filtering
        exampledb=>CREATE PUBLICATION employee_publication_withrows FOR TABLE employee (id, first_name, last_name)  WHERE (department = 'HR');
        CREATE PUBLICATION
    ```
    {: pre}


### Subscriber functions
{: #filter-logical-replication-subscriber-functions}

- **`create_subscription`**

    ```sh
    Arguments:
        subscription_name   Unique name to create the subscription channel with
        host_ip             Publisher hostname or public IP address
        port                Port number publisher is running on
        username            `admin` user created on the publisher
        password            Password of the `admin` user on the publisher
        db_name             The name of the database to be replicated
        publisher_name      The name of publisher channel on the publisher
    
    Usage:
        exampledb=> SELECT create_subscription('subs1','130.215.223.184','5432','password','admin','exampledb','my_publication');
    ```
    {: pre}
  
     When you opt for the default setting `copy_data = true` (an optional parameter, available since PostgreSQL 17) in the subscription parameters, the publisher will only transfer the existing data that matches the row filters specified in the publication(s). This ensures that the initial data replicated to the subscriber adheres strictly to the specified filtering conditions, avoiding the inclusion of any records that fall outside the criteria.
{: .note}
