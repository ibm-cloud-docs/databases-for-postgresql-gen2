---
copyright:
  years: 2019, 2022
lastupdated: "2022-07-05"

keywords: postgresql, databases, icu, postgres icu

subcollection: databases-for-postgresql

---

{:external: .external target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{{site.data.keyword.attribute-definition-list}}

# PostgreSQL ICU Support
{: #postgres-icu}

[ICU](http://site.icu-project.org/){: .external} (International Components for Unicode) uses C/C++ and Java libraries to provide Unicode and Globalization internationalization and localization facilities, including collation. ICU provides stability to your database by preventing index corruption that is caused by operating system collation changes. 

{{site.data.keyword.databases-for-postgresql_full}} supports ICU. To use ICU, your tables need to be created with ICU support, as outlined in the [PostgreSQL Collation Support documentation](https://www.postgresql.org/docs/current/collation.html){: .external}. 

ICU-based collations are offered alongside the `libc` collations. `libc` uses the locales that are provided by the operating system C library and the locales that most tools provided by the operating system use. Building with ICU support does not remove `libc` collation support. For more information, see [PostgreSQL Collation Support](https://www.postgresql.org/docs/current/collation.html){: .external}.

For example, you might have used a command that looks like this. 

`CREATE TABLE ... (... x text COLLATE "en_US" ...)`,

You might now use a command like this.

`CREATE TABLE ... (... x text COLLATE "en-x-icu" ...)`
