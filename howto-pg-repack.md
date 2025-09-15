---

copyright:
  years: 2025
lastupdated: "2025-09-15"

keywords:  postgresql, databases, postgresql extensions, postgres extensions, ibm_extension , pg_repack

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Implementing and utilizing pg_repack
{: #pgrepack}

pg_repack is an extension for your {{site.data.keyword.databases-for-postgresql_full}} that provides functions to reorganize tables and indexes online without requiring table-locking. Its primary function is to reduce table size and improve performance by eliminating bloat.

Bloat typically occurs in PostgreSQL when rows in a table are updated or deleted, leaving space that isn't reused. Over time, this can cause tables to take up more disk space than necessary, leading to decreased performance due to increased disk I/O and memory usage.

pg_repack is supported by PostgreSQL version >9.5.

Run the following query to enable the extension:

```sh
CREATE EXTENSION pg_repack;
```
{: .codeblock}

## Requirements
{: #pgrepack-requirements}

1. The presence of a PRIMARY KEY or a NOT NULL UNIQUE constraint on the table is mandatory.
2. An additional amount of disk space equal to twice the current size is required.

Use the following query to check the table size:

```sh
SELECT schemaname as schemaname,
    relname AS table_name,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM
    pg_catalog.pg_statio_user_tables where relname='<tablename>';
```
{: .codeblock}

3. The pg_repack on the client needs to be compatible with the pg_repack on the server.

Use the following query to view the installed extension and its version:

```sh
SELECT extname AS extension_name,
       extversion AS version,
       nspname AS schema
FROM pg_extension
JOIN pg_namespace ON pg_namespace.oid = pg_extension.extnamespace
ORDER BY extname;
```
{: .codeblock}

## How-to-use-pgrepack
{: #pgrepack-howto}

To run pg_repack with your desired options, use the following syntax:

  ```sh
   pg_repack -k [OPTION]... [DBNAME]

   ```
   {: pre}

- The -k option tells pg_repack to keep the original table if a problem occurs, allowing for safer operation in critical environments.
- Replace [DBNAME] with the name of your PostgreSQL database.
- Common options include:
       
    - `-t` to target a specific table
    - `-U` to specify a user
    - `-h` and `-p` for host and port

After installing the pg_repack extension and verifying that the table has a primary key or a not-null unique constraint, you can repack a table as shown below:

```sh
eg:
exampledb=> create table test(id int);
CREATE TABLE
exampledb=> create table test1(id int PRIMARY KEY, name text);
CREATE TABLE
exampledb=>
exampledb=> CREATE EXTENSION pg_repack;
CREATE EXTENSION
exampledb=>
exampledb=>
exampledb=> SELECT extname AS extension_name,
       extversion AS version,
       nspname AS schema
FROM pg_extension
JOIN pg_namespace ON pg_namespace.oid = pg_extension.extnamespace
ORDER BY extname;
 extension_name | version |    schema
----------------+---------+---------------
 pg_repack      | 1.5.1   | ibm_extension
 plpgsql        | 1.0     | pg_catalog
(2 rows)

----
pg_repack -k --host=localhost --dbname=exampledb --port=31273 --username=test_admin 
INFO: repacking table "public.test1"
```
{: .codeblock}
