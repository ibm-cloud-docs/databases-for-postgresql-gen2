---

copyright:
  years: 2020, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, pgaudit, logging, session, object, pg role, postgresql logging, postgres logging

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Logging with `pgaudit`
{: #pgaudit}

The PostgreSQL Audit Extension (`pgaudit`) provides enablement of session logging for your {{site.data.keyword.databases-for-postgresql_full}} deployments. 

## Session logging
{: #session-logging}

Session logging is off by default. You can enable Session logging parameters that log all activity for sets of audit event types. Session logging is enabled for the whole DB cluster and is either `on` or `off` for a specific event type.

## Event types
{: #event-types}

Session logging is configured per event type. The supported event types across all versions are:

* FUNCTION 
* ROLE
* DDL
* READ
* WRITE
* MISC
* MISC_SET (this additional type is only supported in PostgreSQL 12 or greater)
* NONE (to disable the logging)

For more information, see [`pgaudit`](https://github.com/pgaudit/pgaudit/blob/master/README.md#pgauditlog){: .external}.

## Enabling `pgaudit` session logging
{: #enable-pgaudit-session-logging}

To enable `pgaudit` session logging, connect as the admin user and call the `set_pgaudit_session_logging` function with the appropriate event parameters specified. Session logging is enabled directly in the database and no API or CLI access is provided. 

For example, to enable DDL and ROLE you would call:

```sh
SELECT public.set_pgaudit_session_logging('{ddl, role}');
```
{: .codeblock}

## Enabling `pgaudit` user logging
{: #enable-pgaudit-user-logging}

To enable `pgaudit` user logging, connect as the admin user and call the `set_pgaudit_user_logging` function with the appropriate event parameters specified. User logging is enabled for a specific user instead of all the users in the database.

For example, to enable READ and WRITE, use the following command:

```sh
SELECT public.set_pgaudit_user_logging('{read, write}');
```
{: .codeblock}

## Enabling `pgaudit` database logging
{: #enable-pgaudit-database-logging}

To enable `pgaudit` database specific logging, connect as the admin user and call the `set_pgaudit_database_logging` function with the appropriate event parameters specified. Database logging is enabled for a specific database, in case of multiple databases available in an instance.

For example, to enable DDL and ROLE, use the following command:

```sh
SELECT public.set_pgaudit_database_logging('{ddl, role}');
```
{: .codeblock}

Any subsequent calls replace the existing configuration; they are not additive. For example, a subsequent call to `SELECT public.set_pgaudit_session_logging('{misc}');` logs only `misc` but disable `ddl` and `role`.
{: .note}

## Disabling `pgaudit`
{: #disable-pgaudit}

To disable audit logging, call the same function with `none` specified. For example:

```sh
SELECT public.set_pgaudit_session_logging('{none}');
```
{: .codeblock}

Changing audit levels happens immediately when calling the function without interrupting the database activity.
{: .note}

## Audit logs
{: #pgaudit-logs}

Audit events appear in {{site.data.keyword.logs_routing_full}} with the following format:

```sh
LOG: AUDIT: SESSION,1,1,DDL,CREATE TABLE,,,create table f2 (id int);,<not logged>
```

Form more information, see [`pgaudit` format](https://github.com/pgaudit/pgaudit/blob/master/README.md#format){: .external}. 

If you want to see the current log level, you can run the command:

```sh
show pgaudit.log;
```
{: .codeblock}

