---

copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords: troubleshooting for PostgreSQL, query history, slow queries

subcollection: databases-for-postgresql-gen2

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# PostgreSQL queries
{: #pg-queries}
{: troubleshoot}
{: support}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the  [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

PostgreSQL queries are commands used to retrieve data from a table in a relational database management system. PostgreSQL uses the standard SQL query language.

## How do I track query history?
{: #troubleshoot-long-term-query-history}
{: troubleshoot}
{: support}

I want long-term historical tracking of queries, which helps me diagnose issues that I may be having with slow queries.
{: shortdesc}

Your {{site.data.keyword.databases-for-postgresql}} deployment is experiencing slow queries. How can I track query history to find the root cause?
{: tsSymptoms}

You are experiencing slow queries.
{: tsCauses}

- LogDNA is a log management service that can be integrated with {{site.data.keyword.databases-for}} to collect, analyze, and store logs. Use the LogDNA timeline feature to see how often a search term appears in the log over time. The timeline typically shows log events as bars or markers that are distributed along the time axis. The length or height of the bars or markers can represent the event frequency or other parameters, depending on the timeline visualization. A long query by itself isn't a problem. A series of them can be, and identifying the actual duration can also help.
- Enable [`pg_stat_statements`](https://www.postgresql.org/docs/14/pgstatstatements.html){: external} to identify bottlenecks and areas for optimization. `pg_stat_statements` is a PostgreSQL extension that tracks the planning and execution statistics of all SQL statements that are executed by a server.
- Enable [`logminduration_statement`](https://www.postgresql.org/docs/14/runtime-config-logging.html){: external}, a configuration parameter in PostgreSQL that determines the minimum length of time after which queries should be logged. By setting an appropriate value, you can configure PostgreSQL to log queries that exceed a certain execution time. Adjust the value based on your requirements.
- For more information, see [Changing your {{site.data.keyword.databases-for-postgresql}} Configuration](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration).
{: tsResolve}

## How do I keep slow query logging from exposing sensitive data?
{: #troubleshoot-slow-query-logging}
{: troubleshoot}
{: support}

Logging slow PostgreSQL queries can result in exposing sensitive data into the logs.
{: shortdesc}

Your {{site.data.keyword.databases-for-postgresql}} deployment is logging slow queries and you worry that this will result in sensitive data being exposed.
{: tsSymptoms}

PostgreSQL queries can contain sensitive data such as personal information, passwords, or confidential business data. If these sensitive details are included in a query, they may end up in the log files.
{: tsCauses}

If you worry about exposing sensitive information in your PosgreSQL logs, do the following:
- Use the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5){: external} to adjust the “100 milliseconds” default threshold to a much larger value. Doing so will disable the entry from being written to the logs.
- Adjust your queries to contain hashed passwords instead of cleartext passwords.
    Updating your queries with hashed passwords requires changes to your application.
    {: note}
{: tsResolve}
