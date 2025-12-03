---

copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords:  postgresql, databases, postgresql extensions, postgres extensions, ibm_extension , pg_vector

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Exploring vector-based search in {{site.data.keyword.databases-for-postgresql}} using pgvector
{: #pgvector}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

[pgVector](https://github.com/pgvector/pgvector?tab=readme-ov-file#pgvector){: external} is an extension for your {{site.data.keyword.databases-for-postgresql_full}} that adds support for vector similarity search. It allows you to store and query vectors (arrays of numbers) and to calculate their similarity to each other. This is particularly useful in machine learning applications where you might want to find vectors (for instance, image features or word embeddings) that are similar to a given vector.

pgVector is upported on PostgreSQL version 15 and above.

## Defining and managing vector columns
{: #how-to-use-vector}

To use pgVector to store and perform similarity searches on vector data in {{site.data.keyword.databases-for-postgresql}}, perform the following steps:

1. Use the query below to enable the extension:

    ```sh
    CREATE EXTENSION pg_vector;
    ```
    {: codeblock}

2. You can create a table with a vector column by specifying the dimension of the vectors to be stored, as in the following query:

    ```sh
    eg: 
    exampledb=# CREATE TABLE sample_image (id SERIAL PRIMARY KEY, embedded_vector vector(5));
    CREATE TABLE
    ```
    {: codeblock}

3. Insert data entries and fetch them using a SELECT statement.
   
    ```sh
    eg: 
    exampledb=# INSERT INTO sample_image (embedded_vector) values ('[1,2,3,4,5]');
    INSERT 0 1
    exampledb=# INSERT INTO sample_image (embedded_vector) values (array[0.1, 0.2, 0.3, 0.4, 0.5]);
    INSERT 0 1
   

    exampledb=# select  * from sample_image;
    id |    embedded_vector
    ----+-----------------------
    1 | [1,2,3,4,5]
    2 | [0.1,0.2,0.3,0.4,0.5]
    (2 rows)

    ``` 
    {: codeblock}

4. To improve search performance, create an index on the vector column.

    [Hierarchical Navigable Small World](https://github.com/pgvector/pgvector?tab=readme-ov-file#indexing){: external} (HNSW) and Inverse Vertex File Flat (IVFFlat) are indexing methods used in the context of vector similarity search, commonly employed in applications involving machine learning, data mining, or information retrieval.

    ```sh
    exampledb=# CREATE INDEX sample_image_embedded_vector_idx on sample_image using hnsw(embedded_vector vector_l2_ops);
    CREATE INDEX
    ```
    {: codeblock}

5. There are various operators available in pgvector that can help you perform nearest neighbor searches efficiently. For more information on supported operators and usage examples, see [the pgvector documentation](https://github.com/pgvector/pgvector?tab=readme-ov-file#querying) {: external}.

    ```sh
    exampledb=# select * from sample_image ORDER BY embedded_vector <=>'[0.1,0.2,0.3,0.4,0.5]'::vector ;
    id |    embedded_vector
    ----+-----------------------
    2 | [0.1,0.2,0.3,0.4,0.5]
    1 | [1,2,3,4,5]
    (2 rows)
    ```
    
