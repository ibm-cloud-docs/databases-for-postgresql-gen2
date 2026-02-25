---
copyright:
  years: 2026
lastupdated: "2026-02-25"

keywords: postgresql, databases, postgresql extensions, postgres extensions, ibm_extension , gen2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Managing PostgreSQL extensions
{: #extensions}

In PostgreSQL, extensions are modules that supply extra functions, operators, or types. Many extensions are available in {{site.data.keyword.databases-for-postgresql_full}}. To use them, connect to your deployment [with `psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql).

## Listing installed extensions
{: #listing-installed-extensions}

Get a list of all the extensions installed on a database by using the `\dx` command.

For example, the output for `\dx` when run on the {{site.data.keyword.databases-for-postgresql}} default database shows the only installed extension.

```sh
postgres=> \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(1 row)
```

## Installing extensions
{: #installing-extensions}

To install an extension on to a database use [`CREATE EXTENSION`](https://www.postgresql.org/docs/current/static/sql-createextension.html){: .external}. For example, to install `pg_stat_statements` on the `ibmclouddb` database, use the following command:

```sh
postgres=> CREATE EXTENSION pg_stat_statements;
CREATE EXTENSION
```
{: pre}

Extensions are installed into the read-only `ibm_extension` schema. The schema is part of the `search_path` so extension objects do not need to be qualified with a schema.
The change from `public` schema to `ibm_extension` schema is necessary to protect the security and integrity of your data.

If you run the `\dx` command after installing an extension, it appears in the table.

```sh
postgres=> \dx
                                     List of installed extensions
        Name        | Version |   Schema      |                        Description
--------------------+---------+---------------+-----------------------------------------------------------
 pg_stat_statements | 1.5     | ibm_extension | track execution statistics of all SQL statements executed
 plpgsql            | 1.0     | pg_catalog    | PL/pgSQL procedural language
(2 rows)
```

Database extensions in PostgreSQL are managed per database. If you have multiple databases that you need to install an extension on, run the `CREATE` command on each database.

## Upgrading extensions
{: #upgrading-extensions}

If there is a newer version of an extension available than the one you currently have installed, use the `ALTER EXTENSION` to upgrade it.

## Extension-specific notes
{: #extensions-specific-notes}

### pg_repack
{: #pg_repack}

- [The `pg_repack` documentation](http://reorg.github.io/pg_repack/){: .external}
- When you run the `pg_repack` command, pass the -k flag in to bypass the check for superuser. See the following example:

   ```sh
   pg_repack -k [OPTION]... [DBNAME]
   ```
   {: pre}

- For `pg_repack` to run reliably, your deployment should be on PostgreSQL 9.6 and above.
- Any user can run `pg_repack`, but the command is only able to repack a table that they have permissions on.
- `pg_repack` needs to take an exclusive lock on objects it is reorganizing at the end of the reorganization. If it can't get this lock after a certain period, it cancels all conflicting queries. If it can't do so, the reorg fails. By default, only the admin user on PostgreSQL 9.6 and greater is able to cancel conflicting queries. To expose the ability to cancel queries to other database users, grant the `pg_signal_backend` role [from the admin user](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-admin-user).


### pgaudit
{: #pgaudit}

- `pgaudit` libraries are preloaded and do not require execution of `create extension pgaudit`. For more information, see [Logging with pgAudit](/docs/databases-for-postgresql?topic=databases-for-postgresql-pgaudit) to enable pgaudit logs. 


### pgvector
{: #pgvector}

-  To add the `pgvector` extension to your deployment, use the `create extension vector` command.
-  Important: `pgvector` requires PostgreSQL version 15 or higher.


## Available extensions
{: #available-extensions}

See the following list of all available extensions. For the latest list of available extensions on your deployment, use `SELECT name FROM pg_available_extensions;` in `psql`.

```sh
postgres=> SELECT name FROM pg_available_extensions order by 1;
```
{: pre}


## PostgreSQL Extensions

| Name | Default Version | Installed Version | Comment |
|------|-----------------|-------------------|---------|
| address_standardizer | 3.6.2 | | Used to parse an address into constituent elements. Generally used to support geocoding address normalization step. |
| address_standardizer_data_us | 3.6.2 | | Address Standardizer US dataset example |
| amcheck | 1.5 | | functions for verifying relation integrity |
| anon | 2.5.1 | | Anonymization & Data Masking for PostgreSQL |
| autoinc | 1.0 | | functions for autoincrementing fields |
| bloom | 1.0 | | bloom access method - signature file based index |
| btree_gin | 1.3 | | support for indexing common datatypes in GIN |
| btree_gist | 1.8 | | support for indexing common datatypes in GiST |
| citext | 1.8 | | data type for case-insensitive character strings |
| cube | 1.5 | | data type for multidimensional cubes |
| dblink | 1.2 | | connect to other PostgreSQL databases from within a database |
| dict_int | 1.0 | | text search dictionary template for integers |
| dict_xsyn | 1.0 | | text search dictionary template for extended synonym processing |
| earthdistance | 1.2 | | calculate great-circle distances on the surface of the Earth |
| file_fdw | 1.0 | | foreign-data wrapper for flat file access |
| fuzzystrmatch | 1.2 | | determine similarities and distance between strings |
| hstore | 1.8 | | data type for storing sets of (key, value) pairs |
| hypopg | 1.4.2 | | Hypothetical indexes for PostgreSQL |
| insert_username | 1.0 | | functions for tracking who changed a table |
| intagg | 1.1 | | integer aggregator and enumerator (obsolete) |
| intarray | 1.5 | | functions, operators, and index support for 1-D arrays of integers |
| isn | 1.3 | | data types for international product numbering standards |
| lo | 1.2 | | Large Object maintenance |
| ltree | 1.3 | | data type for hierarchical tree-like structures |
| moddatetime | 1.0 | | functions for tracking last modification time |
| orafce | 4.16 | | Functions and operators that emulate a subset of functions and packages from the Oracle RDBMS |
| pageinspect | 1.13 | | inspect the contents of database pages at a low level |
| pg_buffercache | 1.6 | | examine the shared buffer cache |
| pg_cron | 1.6 | | Job scheduler for PostgreSQL |
| pg_freespacemap | 1.3 | | examine the free space map (FSM) |
| pg_hint_plan | 1.8.0 | | optimizer hints for PostgreSQL |
| pg_logicalinspect | 1.0 | | functions to inspect logical decoding components |
| pg_partman | 5.4.0 | | Extension to manage partitioned tables by time or ID |
| pg_prewarm | 1.2 | | prewarm relation data |
| pg_repack | 1.5.3 | | Reorganize tables in PostgreSQL databases with minimal locks |
| pg_stat_monitor | 2.3 | | The pg_stat_monitor is a PostgreSQL Query Performance Monitoring tool, based on PostgreSQL contrib module pg_stat_statements. pg_stat_monitor provides aggregated statistics, client information, plan details including plan, and histogram information. |
| pg_stat_statements | 1.12 | | track planning and execution statistics of all SQL statements executed |
| pg_surgery | 1.0 | | extension to perform surgery on a damaged relation |
| pg_textsearch | 0.4.1 | | Full-text search with BM25 ranking |
| pg_trgm | 1.6 | | text similarity measurement and index searching based on trigrams |
| pg_visibility | 1.2 | | examine the visibility map (VM) and page-level visibility info |
| pg_walinspect | 1.1 | | functions to inspect contents of PostgreSQL Write-Ahead Log |
| pgaudit | 18.0 | | provides auditing functionality |
| pgcrypto | 1.4 | | cryptographic functions |
| pgrouting | 3.8.0 | | pgRouting Extension |
| pgrowlocks | 1.2 | | show row-level locking information |
| pgstattuple | 1.5 | | show tuple-level statistics |
| plpgsql | 1.0 | 1.0 | PL/pgSQL procedural language |
| postgis | 3.6.2 | 3.6.2 | PostGIS geometry and geography spatial types and functions |
| postgis_raster | 3.6.2 | | PostGIS raster types and functions |
| postgis_tiger_geocoder | 3.6.2 | | PostGIS tiger geocoder and reverse geocoder |
| postgis_topology | 3.6.2 | | PostGIS topology spatial types and functions |
| postgres_fdw | 1.2 | | foreign-data wrapper for remote PostgreSQL servers |
| refint | 1.0 | | functions for implementing referential integrity (obsolete) |
| seg | 1.4 | | data type for representing line segments or floating-point intervals |
| sslinfo | 1.2 | | information about SSL certificates |
| tablefunc | 1.0 | | functions that manipulate whole tables, including crosstab |
| tcn | 1.0 | | Triggered change notifications |
| temporal_tables | 1.2.2 | | temporal tables |
| tsm_system_rows | 1.0 | | TABLESAMPLE method which accepts number of rows as a limit |
| tsm_system_time | 1.0 | | TABLESAMPLE method which accepts time in milliseconds as a limit |
| unaccent | 1.1 | | text search dictionary that removes accents |
| uuid-ossp | 1.1 | | generate universally unique identifiers (UUIDs) |
| vector | 0.8.1 | 0.8.1 | vector data type and ivfflat and hnsw access methods |
| vectorscale | 0.9.0 | | diskann access method for vector search |
| xml2 | 1.2 | | XPath querying and XSLT |
