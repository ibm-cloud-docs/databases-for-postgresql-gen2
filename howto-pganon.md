---

copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords: postgresql, databases, PostgreSQL Anonymizer, Postgresql data masking, GDPR compliance PostgreSQL, Static Masking, Dynamic Masking, Anonymous Dumps, Generalization

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Data masking for {{site.data.keyword.databases-for-postgresql}}
{: #data-masking}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

{{site.data.keyword.databases-for-postgresql}} Anonymizer is a PostgreSQL extension designed to anonymize data by masking or replacing personally identifiable and sensitive business information in a {{site.data.keyword.databases-for-postgresql}} database.

The extension supports the following masking strategies, each suitable for different use cases:

- **Static masking**: Remove the PII according to the rules.
- **Dynamic masking**: Hide PII only for the masked users.
- **Anonymous dumps**: Simply export the masked data into an SQL file.
- **Generalization**: Replaces specific values with broader, less precise categories or range.

## Static masking
{: #static-masking}

Static masking is a data anonymization technique that irreversibly replaces sensitive information with synthetically generated,
format-preserving values. This process ensures the complete and permanent removal of the original data, thereby eliminating
the possibility of re-identification or reconstruction.
For more information, see [Static masking](https://postgresql-anonymizer.readthedocs.io/en/stable/tutorials/1-static_masking).

### Enabling static masking
{: #enabling-static-masking}

1. Install the Anonymizer Extension.

    ```sh
    CREATE EXTENSION IF NOT EXISTS anon;
    ```
    {: pre}

2. Initialize the Anonymizer.

    ```sh
    select public.run_anon_init();
    ```
    {: pre}

3. Define masking rules.

    ```sh
    SECURITY LABEL FOR anon
      ON COLUMN <table_name>.<column_name>
      IS 'MASKED WITH FUNCTION anon.<masking_function>()';
    ```
    {: pre}

4. Verify the applied masking rules.

    ```sh
    SELECT objoid::regclass, provider, label
    FROM pg_seclabel
    WHERE provider = 'anon' AND objoid = '<table_name>'::regclass;
    ```
    {: pre}

5. Apply the rules permanently. below command permanently replaces the original data in the specified table with the masked data.

    ```sh
    SELECT anon.anonymize_table('<table_name>');
    ```
    {: pre}

Static masking is a slow process. The principle of static masking is to update all lines of all tables containing at
least one masked column. This means that PostgreSQL will rewrite all the data on disk.
Depending on the database size, the hardware, and the instance config, it may be faster to export the anonymized data and reload it into the database.
{: note}

## Dynamic masking
{: #dynamic-masking}

Dynamic masking allows the database owner to obscure sensitive data for specific users,
while permitting authorized users to access and modify the original, unmasked values.
For more information, see [Dynamic masking](https://postgresql-anonymizer.readthedocs.io/en/stable/tutorials/2-dynamic_masking/).

### Enabling dynamic masking
{: #enabling-dynamic-masking}

1. Install the Anonymizer Extension.

    ```sh
    CREATE EXTENSION IF NOT EXISTS anon;
    ```
    {: pre}

2. Initialize the Anonymizer.

    ```sh
    select public.run_anon_init();
    ```
    {: pre}

3. Activate the dynamic masking engine.

    ```sh
    show anon.transparent_dynamic_masking;

    select public.enable_dynamic_masking ('<database_name>');
    ```
    {: pre}

    Restart your session if anon.transparent_dynamic_masking is showing as off.
    {: note}

4. Create the role for masked access.

    ```sh
    CREATE ROLE <role_name> LOGIN;
    ```
    {: pre}

5. Mark the role as a masked user.

    ```sh
      SECURITY LABEL FOR anon ON ROLE <role_name> IS 'MASKED';
    ```
    {: pre}

6. Declare the masking rules.

    ```sh
    SECURITY LABEL FOR anon
      ON COLUMN <table_name>.<column_name>
      IS 'MASKED WITH FUNCTION anon.<masking_function>()';
    ```
    {: pre}

7. Verify the masking rules.

    ```sh
    SELECT objoid::regclass, provider, label
    FROM pg_seclabel
    WHERE provider = 'anon' AND objoid = '<table_name>'::regclass;
    ```
    {: pre}

8. Grant Select access on masked tables.

    ```sh
    grant select on <schema_name>.<table_name> to <role_name>;
    ```
    {: pre}

9. Connect to the database using the masked role, and query the table. You should see the masked data as per the rules defined.

    ```sh
    select * from <table_name>;
    ```
    {: pre}

Notes:

- The anonymizer masks data based on who is querying.
- Masked roles should not be allowed to insert, update, or delete data.
- You can mask table in multiple schemas.
- A masking rule may break data integrity. For instance, you can mask a column having a UNIQUE constraint with the value NULL. It is up to you to decide whether or not the mask users need data integrity.
- Masked roles are not allowed to use EXPLAIN.

## Anonymous dumps
{: #anonymous-dumps}

An anonymous dump refers to the process of exporting a database that contains anonymized or de-identified data,
rather than the original sensitive values. This is typically done to safely share, migrate, or analyze data without
exposing personally identifiable information (PII) or sensitive attributes. For more information, see [Anonymous dumps](https://postgresql-anonymizer.readthedocs.io/en/latest/anonymous_dumps/).

### Enabling anonymous dumps
{: #enabling-anonymous-dumps}

1. Install the Anonymizer Extension.

    ```sh
    CREATE EXTENSION IF NOT EXISTS anon;
    ```
    {: pre}

2. Initialize the Anonymizer.

    ```sh
    select public.run_anon_init();
    ```
    {: pre}

3. Create a dedicated schema for custom masking functions.

    ```sh
     CREATE SCHEMA IF NOT EXISTS <schema_name>;
    ```
    {: pre}

4. Declare the schema as trusted. This allows the anonymizer to recognize custom masking functions defined in this schema.

    ```sh
    SECURITY LABEL FOR anon ON SCHEMA <schema_name> IS 'TRUSTED';
    ```
    {: pre}

5. Define a custom masking function, as in the following example.

    ```sh
    CREATE OR REPLACE FUNCTION <schema_name>.remove_content(j JSONB)
    RETURNS JSONB
    AS $func$
      SELECT j - ARRAY['content']
    $func$
    LANGUAGE SQL;
    ```
    {: pre}

6. Create a masking rule.

    ```sh
    SECURITY LABEL FOR anon
      ON COLUMN <table_name>.<column_name>
      IS 'MASKED WITH FUNCTION anon.<custom_masking_function>()';
    ```
    {: pre}

7. Create a role for exporting masked data.

    ```sh
     CREATE ROLE <role_name> LOGIN PASSWORD '<enter_password>';
    ```
    {: pre}

8. Label the role as a masked role.

    ```sh
    SECURITY LABEL FOR anon ON ROLE <schema_name> IS 'MASKED';
    ```
    {: pre}

9. Enable masking for role.

    ```sh
    select  public.enable_masking_for_dump('<role_name>','<schema_name>');
    ```
    {: pre}

10. Grant access to the masked schema.

    ```sh
    GRANT USAGE ON SCHEMA <masked_schema_name>> TO <masked_role>;
    ```
    {: pre}

11. Grant Select access on masked tables.

    ```sh
    grant select ON <schema_name>.<table_name> to <masked_role>;
    ```
    {: pre}

12. Export masked data using pg_dump as the masked role.

    ```sh
    pg_dump -U <masked_user> -d <database_name> -f <dump_file_name>
    ```
    {: pre}

## Generalization
{: #generalization}

The main idea of generalization is to blur the original data.
For more information, see [Generalization](https://postgresql-anonymizer.readthedocs.io/en/stable/tutorials/4-generalization/#).

### Enabling generalization
{: #enabling-generalization}

1. Install the anonymizer extension.

    ```sh
    CREATE EXTENSION IF NOT EXISTS anon;
    ```
    {: pre}

2. Initialize the anonymizer.

    ```sh
    select public.run_anon_init();
    ```
    {: pre}

3. Create a view using generalization functions.

    ```sh
    CREATE MATERIALIZED VIEW <view_name> AS
    SELECT
        anon.generalize_daterange(<date_column_1>, 'month') AS <generalized_date_1>,
        anon.generalize_daterange(<date_column_2>, 'month') AS <generalized_date_2>
    FROM <source_table>;
    ```
    {: pre}

4. Declare indirect identifiers.

    ```sh
    SECURITY LABEL FOR k_anonymity
      ON COLUMN <view_name>.<generalized_date_1> IS 'INDIRECT IDENTIFIER';
    
    SECURITY LABEL FOR k_anonymity
      ON COLUMN <view_name>.<generalized_date_2> IS 'INDIRECT IDENTIFIER';
    ```
    {: pre}

5. Grant access to k-anonymity.

    ```sh
    select public.k_anonymity();
    ```
    {: pre}

6. Run the k-anonymity check on the view or table.

    ```sh
    SELECT anon.k_anonymity('<view_name/table_name>');
    ```
    {: pre}

## Unmask a role
{: #unmask-role}

Use the following command to unmask a role.

```sh
SECURITY LABEL FOR anon ON ROLE <role_name> IS NULL;
```
{: pre}

## Disable masking rule
{: #disable-masking}

Use the following command to disable the masking role.

```sh
SECURITY LABEL FOR anon
  ON COLUMN <table_name>.<column_name>
  IS NULL;
```
{: pre}
