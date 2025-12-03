---
copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords: postgresql, databases, postgresql extensions, postgres extensions, ibm_extension

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Managing PostgreSQL extensions
{: #extensions}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

In PostgreSQL, extensions are modules that supply extra functions, operators, or types. Many extensions are available in {{site.data.keyword.databases-for-postgresql_full}}. For the following examples, use your service credentials and [connect to `psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql) before proceeding.

## Listing installed extensions
{: #listing-installed-extensions}

Get a list of all the extensions installed on a database by using the `\dx` command.

For example, the output for `\dx` when run on the {{site.data.keyword.databases-for-postgresql}} default database shows the only installed extension.

```sh
ibmclouddb=> \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(1 row)
```
{: pre}

## Installing extensions
{: #installing-extensions}

To install an extension on to a database, use  [`CREATE EXTENSION`](https://www.postgresql.org/docs/current/static/sql-createextension.html){: .external}. For example, to install `pg_stat_statements` on the `ibmclouddb` database, use the following command:

```sh
ibmclouddb=> CREATE EXTENSION pg_stat_statements;
CREATE EXTENSION
```
{: pre}

Extensions are installed into the `public` schema by default. Because the `search_path` automatically includes this schema, you can reference (or "call") extension objects without needing to specify the schema qualifier.

If you run the `\dx` command after installing an extension, it appears in the table.

```sh
ibmclouddb=> \dx
                                     List of installed extensions
        Name        | Version |   Schema      |                        Description
--------------------+---------+---------------+-----------------------------------------------------------
 pg_stat_statements | 1.5     | public        | track execution statistics of all SQL statements executed
 plpgsql            | 1.0     | pg_catalog    | PL/pgSQL procedural language
(2 rows)
```
{: pre}

Database extensions in PostgreSQL are managed per database. If you have multiple databases that you need to install an extension on, run the `CREATE` command on each database.

## Upgrading extensions
{: #upgrading-extensions}

When a newer version of your installed extension is available, you can upgrade it by running `ALTER EXTENSION`. 

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

- For `pg_repack` to run reliably, your deployment should be on the most recent version of PostgreSQL.
- Any user can run `pg_repack`, but the command is only able to repack a table that they have permissions on.
- `pg_repack` needs to take an exclusive lock on objects it is reorganizing at the end of the reorganization. If it can't get this lock after a certain period, it cancels all conflicting queries. If it can't do so, the reorg fails. To expose the ability to cancel queries to other database users, grant the `pg_signal_backend` role [from the Manager user](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-admin-user).


## Available extensions
{: #available-extensions}

See the following list of all available extensions. For the latest list of available extensions on your deployment, execute the query `SELECT name FROM pg_available_extensions;` in `psql`.

```sh
ibmclouddb=> SELECT name FROM pg_available_extensions order by 1;
```
{: pre}

```sh
List of currently available extensions on VPC

 amcheck                              | functions for verifying relation integrity
 autoinc                              | functions for autoincrementing fields
 bloom                                | bloom access method - signature file based index
 btree_gin                            | support for indexing common datatypes in GIN
 btree_gist                           | support for indexing common datatypes in GiST
 citext                               | data type for case-insensitive character strings
 cube                                 | data type for multidimensional cubes
 dblink                               | connect to other PostgreSQL databases from within a database
 dict_int                             | text search dictionary template for integers
 dict_xsyn                            | text search dictionary template for extended synonym processing
 earthdistance                        | calculate great-circle distances on the surface of the Earth
 file_fdw                             | foreign-data wrapper for flat file access
 fuzzystrmatch                        | determine similarities and distance between strings
 hstore                               | data type for storing sets of (key, value) pairs
 insert_username                      | functions for tracking who changed a table
 intagg                               | integer aggregator and enumerator (obsolete)
 intarray                             | functions, operators, and index support for 1-D arrays of integers
 isn                                  | data types for international product numbering standards
 lo                                   | Large Object maintenance
 ltree                                | data type for hierarchical tree-like structures
 moddatetime                          | functions for tracking last modification time
 pageinspect                          | inspect the contents of database pages at a low level
 pg_buffercache                       | examine the shared buffer cache
 pg_freespacemap                      | examine the free space map (FSM)
 pg_logicalinspect                    | functions to inspect logical decoding components
 pg_prewarm                           | prewarm relation data
 pg_stat_statements                   | track planning and execution statistics of all SQL statements executed
 pg_surgery                           | extension to perform surgery on a damaged relation
 pg_trgm                              | text similarity measurement and index searching based on trigrams
 pg_visibility                        | examine the visibility map (VM) and page-level visibility info
 pg_walinspect                        | functions to inspect contents of PostgreSQL Write-Ahead Log
 pgcrypto                             | cryptographic functions
 pgrowlocks                           | show row-level locking information
 pgstattuple                          | show tuple-level statistics
 plpgsql                              | PL/pgSQL procedural language
 postgres_fdw                         | foreign-data wrapper for remote PostgreSQL servers
 refint                               | functions for implementing referential integrity (obsolete)
 seg                                  | data type for representing line segments or floating-point intervals
 sslinfo                              | information about SSL certificates
 tablefunc                            | functions that manipulate whole tables, including crosstab
 tcn                                  | Triggered change notifications
 tsm_system_rows                      | TABLESAMPLE method which accepts number of rows as a limit
 tsm_system_time                      | TABLESAMPLE method which accepts time in milliseconds as a limit
 unaccent                             | text search dictionary that removes accents
 xml2                                 | XPath querying and XSLT
 ```
{: pre}
