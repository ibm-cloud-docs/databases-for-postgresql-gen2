---
copyright:
  years: 2017, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, postgres admin user, postgresql admin user, pg_dump, postgres migration, postgresql migration

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Migrating to {{site.data.keyword.databases-for-postgresql}}
{: #migrating}

Various options exist to migrate data from existing PostgreSQL databases to {{site.data.keyword.databases-for-postgresql_full}}. We focus on the simplest and most effective. To get started, you need PostgreSQL installed locally so you have the `psql` and `pg_dump` tools. And while not strictly required, the {{site.data.keyword.databases-for}} CLI makes it easier to connect and restore to a new {{site.data.keyword.databases-for-postgresql}} deployment. 

## pg_dump
{: #pg_dump}

On your source database run `pg_dump` to create an SQL file, which can be used to re-create the database. At a minimum, `pg_dump` takes a hostname (`-h` flag), port number (`-p` flag), database name (`-d` flag), username (`-U` flag), and a file (or directory name) to write the dump to (`-f` flag). 

For example, the following command dumps the PostgreSQL "compose" database that is hosted on sl-eu-lon-2-portal.4.dblayer.com, port 17980, using the admin user and save the results in `dump.sql`.

```sh
pg_dump -h sl-eu-lon-2-portal.4.dblayer.com -p 17980 -d compose -U admin -f dump.sql
```
{: pre}

The `pg_dump` command has many options and it is recommended that you [consult the official documentation](https://www.postgresql.org/docs/current/backup-dump.html){: .external} and [command reference](https://www.postgresql.org/docs/current/app-pgdump.html){: .external} for a fuller view of its capabilities.

Using pg_dumpall will causes permission issues, as it tries to include system tables in the operation, which is not supported. As documented, use pg_dump and specify tables that are user-specific.
{: .tip}

## Restoring pg_dump's output
{: #restore-pg_dump-output}

The resulting output of `pg_dump` can then be uploaded into a new {{site.data.keyword.databases-for-postgresql}} deployment. As the output is SQL, it can simply be sent to the database through the `psql` command. We recommend that imports be performed with the admin user. 

See the [Connecting with `psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql) for details on how to connect as admin by using `psql`. To connect with the `psql` command, you need the admin user's connection string and the TLS certificate. The certificate needs to be decoded from the base64 and stored as an arbitrary local file. To import the previously created `dump.sql` into a database deployment named `example-psql`, the `psql` command can be called with `-f dump.sql` as a parameter. The parameter tells `psql` to read and execute the SQL statements in the file. The command looks something like:

```sh
PGPASSWORD=yourpasswordhere PGSSLROOTCERT=cert.crt psql 'host=c7798cf6-e5d2-4513-b17f-3d3fa67d8291.8f7bfd8f3faa4218aec56e069eb46187.databases.appdomain.cloud port=32484 dbname=ibmclouddb user=admin sslmode=verify-full' -f dump.sql
```
{: pre}

As noted in that [Connecting with `psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql) documentation, the {{site.data.keyword.databases-for}} CLI plug-in simplifies connecting. The previous `psql` import can be performed as:

```sh
ibmcloud cdb deployment-connections example-psql -s -- -f dump.sql
```
{: pre}

The command automatically uses the admin user, if no user is specified. It also interactively prompts for the password. The TLS certificate is automatically retrieved and used. The `-s` starts `psql` (or whatever command has been configured) once the details are established from the API. Anything after the `--` is passed to the command.

While the restore process is running, it emits a number of messages about changes it is making to the database deployment.

### Additional migration option by using pg_restore
{: #pg_restore}

For users with a TAR file containing sql and data separately, the command `pg_restore` can be used to migrate your data in addition to the `psql` commands previously noted. An example of the `pg_restore` command is:

```sh
 PGPASSWORD=yourpasswordhere PGSSLROOTCERT=cert.crt pg_restore -h c7798cf6-e5d2-4513-b17f-3d3fa67d8291.8f7bfd8f3faa4218aec56e069eb46187.databases.appdomain.cloud -p 32484 -U admin -F t -d ibmclouddb tarfile.tar
 ```
 {: .pre}

