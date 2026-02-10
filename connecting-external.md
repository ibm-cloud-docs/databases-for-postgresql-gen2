---
copyright:
  years: 2026
lastupdated: "2026-02-10"

keywords: postgresql drivers, python, java, javascript, certificate, postgresql connection string, postgresql connecting external application, postgresql python, Gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Connecting an external application to your PostgreSQL deployment
{: #external-app}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

Your applications and drivers use connection strings to make a connection to {{site.data.keyword.databases-for-postgresql_full}}. The service provides connection strings specifically for drivers and applications. Connection strings are displayed in the *Endpoints* panel of your deployment's *Overview*, and can also be retrieved from the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference ), and the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api).

{{site.data.keyword.databases-for-postgresql}} deployments no longer include a default admin user. Instead, customers create users with 'Manager', 'Writer', or 'Reader' roles through the {{site.data.keyword.cloud}} service credential interface — available via the UI or CLI. This process generates credentials for connecting to the deployment. While these credentials can be used across multiple connections and applications, it is strongly recommended to create dedicated users for each application, tailored to their specific access requirements. For more information, see [Getting connection strings](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-connection-strings).

## Connecting to your PostgreSQL deployment with a language's driver
{: #connect-language-driver}

All the information a driver needs to make a connection to your deployment is in the "postgres" section of your connection strings. The table contains a breakdown for reference.

| Field name | Index | Description |
| ---------- | ----- | ----------- |
| `Type` | | Type of connection - for PostgreSQL, it is "URI". |
| `Scheme` | | Scheme for a URI - for PostgreSQL, it is "postgresql". |
| `Path` | | Path for a URI - for PostgreSQL, it is the database name. The default is `ibmclouddb`. |
| `Authentication` | `Username` | The username that you use to connect. |
| `Authentication` | `Password` | A password for the user - might be shown as `$PASSWORD` |
| `Authentication` | `Method` | How authentication takes place; "direct" authentication is handled by the driver. |
| `Hosts` | `0...` | A hostname and port to connect to. |
| `Composed` | `0...` | A URI combining Scheme, Authentication, Host, and Path. |
| `Certificate` | `Name` | The allocated name for the service proprietary certificate for database deployment. |
| `Certificate` | Base64 | A base64 encoded version of the certificate. |
{: caption="postgres/URI connection information" caption-side="bottom"}

* `0...` indicates that there might be one or more of these entries in an array.

Many PostgreSQL drivers are able to make a connection to your deployment when given the URI-formatted connection string found in the "composed" field of the connection information. For example,

```sh
postgres://ibm_cloud_30399dec_4835_4967_a23d_30587a08d9a8:$PASSWORD@981ac415-5a35-4ac7-b6bb-fb609326dc42.8f7bfd8f3faa4218aec56e069eb46187.databases.appdomain.cloud:32704/ibmclouddb?sslmode=verify-full
```
{: .codeblock}

The following example uses the information from your connection string and the Java driver [`jdbc`](https://jdbc.postgresql.org/documentation/head/index.html){: .external} to connect to your database.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;
import java.util.logging.*;

public class PGConnect {

    private final static Logger LOGGER = Logger.getLogger(Logger.GLOBAL_LOGGER_NAME);

    private Connection connect() {

        final String url = "jdbc:postgresql://host:port/ibmclouddb";

        Properties props = new Properties();
        props.setProperty("user","username");
        props.setProperty("password","mypassword123");
        props.setProperty("ssl","true");
        props.setProperty("sslmode","verify-full");
        props.setProperty("sslrootcert", "/path/to/cert");

        Connection conn = null;
        while (conn == null) {
            try {
                conn = DriverManager.getConnection(url, props);
                System.out.println("Connected to PG");
            } catch (SQLException e) {
                System.out.printf("%s\n", e);
                LOGGER.info("Not connected, retying ...");
            }
        }

        return conn;

    }

    public static void main(String[] args) {

        PGConnect icd = new PGConnect();

        try {
            Connection connection = icd.connect();
            Statement stmt = connection.createStatement();

            ResultSet rs = stmt.executeQuery("SELECT * from pg_database");
            while (rs.next()) {
                System.out.println("DB Name: " + rs.getString(1));
            }

        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }
}
```
{: .codeblock}

The following example uses the information from your connection string and the Python driver [`Psycopg2`](https://wiki.postgresql.org/wiki/Psycopg2_Tutorial){: .external} to connect to your database. This is just a simple connection example, without error handling or retry logic and might not be suitable for production.

```python
import psycopg2

try:
    conn = psycopg2.connect(
      host="hostname.databases.appdomain.cloud",
      port= 31525,
      user="username",
      password="password",
      sslmode="verify-full",
      sslrootcert="/path/to/cert/ca-certificate.crt",
      database="ibmclouddb")
except:
    print("Unable to connect to database")

cur = conn.cursor()
cur.execute("SELECT datname FROM pg_database")
rows = cur.fetchall()

print("List of databases:")
for row in rows:
    print("  ",row[0])
```
{: .codeblock}

The following example uses the information from your connection string and the Node driver [`node-postgres`](https://node-postgres.com/){: .external} to connect to your database.

```java
const pg = require("pg");
const fs = require("fs");

let connectionString = "postgres://<username>:<password>@<host>:<port>/<database>";
let caCert = fs.readFileSync('/path/to/cert');

// set up a client with your PostgreSQL connection string and TLS options
let client = new pg.Client({
    connectionString: connectionString,
    ssl: {
    ca: caCert,
    rejectUnauthorized: true
    }
});


client.connect(function(err) {
    if (err) {
        console.log(err);
        process.exit(1);
    } else {
        // query for the names of the databases
        client.query(
            "SELECT datname FROM pg_database;",
            function(err, result) {
                if (err) {
                    console.log(err);
                }
                // return the names of the databases
                console.log(result.rows);
                client.end();
            }
        );
    }
});
```
{: .codeblock}

To use the `node-postgres driver`, remove the `sslmode` parameter from the deployment's connection string. If not, this parameter overrides the `ssl: {...}` parameters, preventing the CA certificate from properly loading.
{: .note}

## Driver TLS and service proprietary certificate support
{: #tls-certificate-support}

All connections to {{site.data.keyword.databases-for-postgresql}} are TLS 1.2 enabled, so the driver you use to connect needs to be able to support encryption. Your deployment also comes with a service proprietary certificate so the driver can verify the server upon connection.

For more information, see [{{site.data.keyword.databases-for}} Certificates FAQ](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-faq-cert){: external}.


### Using the service proprietary certificate
{: #selfsigned-cert}

1. Copy the certificate information from the *Endpoints* panel or the Base64 field of the connection information.
2. If needed, decode the Base64 string into text.
3. Save the certificate to a file. (You can use the name that is provided or your own file name).
4. Provide the path to the certificate to the driver or client.

![CLI Endpoints panel](images/cli-endpoints-pane.png){: caption="The CLI plug-in information tab" caption-side="bottom"}

### CLI plug-in support for the service proprietary certificate
{: #cli-support}

You can display the decoded certificate for your deployment with the CLI plug-in with the command `ibmcloud cdb deployment-cacert "example-deployment"`. It decodes the base64 into text. Copy and save the command's output to a file and provide the file's path to the driver.

## Other Drivers
{: #other-drivers}

PostgreSQL has a vast array of language drivers. The table covers a few of the most common.

| Language | Driver | Examples |
| ------- | ------- | ------- |
| PHP | `pgsql` | [Link](http://php.net/manual/en/pgsql.examples-basic.php){: .external} |
| Ruby| `ruby-pg` | [Link](https://github.com/ged/ruby-pg){: .external} |
| Ruby on Rails | Rails | [Link](http://edgeguides.rubyonrails.org/configuring.html#configuring-a-postgresql-database){: .external} |
| C# | `ODBC` | [Link](https://wiki.postgresql.org/wiki/Using_Microsoft_.NET_with_the_PostgreSQL_Database_Server_via_ODBC){: .external} |
| Go | `pq` | [Link](https://godoc.org/github.com/lib/pq){: .external} |
| Node | `node-postgres` | [Link](https://node-postgres.com/){: .external} |
{: caption="PostgreSQL drivers" caption-side="bottom"}
