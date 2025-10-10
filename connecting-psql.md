---
copyright:
  years: 2025
lastupdated: "2025-10-10"

keywords: postgresql, databases, psql, postgresql command line, Gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}


# Connecting with `psql`
{: #connecting-psql}

Use `psql` for direct interaction and monitoring of the data structures that are created within the database. `psql` is also useful for testing and monitoring queries and performance, installing and modifying scripts, and other management activities.

Ensure that the user connecting to the deployment has the role [`pg_monitor`](https://www.postgresql.org/docs/10/static/default-roles.html){: .external}, which allows access to PostgreSQL monitoring views and functions.

{{site.data.keyword.databases-for-postgresql}} deployments no longer include a default admin user. Instead, customers create a user with the 'Manager', 'Writer', or 'Reader' role using the {{site.data.keyword.cloud}} service credential interface — via UI or CLI. The process provides credentials for connecting to the deployment.
{: .tip}

## Installing `psql`
{: #installing-psql}

To use `psql`, the PostgreSQL client tools need to be installed on the local system. They can be installed with the full PostgreSQL package that is provided from [postgresql.org](https://www.postgresql.org/download/){: .external}, as a [package from your operating system's package manager](https://www.ibm.com/blog/postgresql-tips-installing-the-postgresql-client/){: .external}.

For more information about `psql`, see the [PostgreSQL documentation](https://www.postgresql.org/docs/current/static/app-psql.html){: .external}.

Most instructions for installing the PostgreSQL tools assume you want the database installed too. It's a reasonable assumption if you're dealing with users who don't have access to PostgreSQL in the cloud or on a remote server.

Here are steps for installing `psql` as a stand-alone tool on macOS, Linux, and Windows. 

### Installing `psql` on macOS with Homebrew
{: #installing-psql-macos}

We recommend [Homebrew](https://brew.sh/){: external} as a package manager for macOS. With Homebrew, you are able to install numerous applications, usually with the programs available in `/usr/local/bin`. Homebrew's package for the PostgreSQL client tools is the `libpq` package. Brew makes it easy to install:

```sh
brew install libpq
```
{: pre}

There's a small catch though: `libpq` won't install itself in the `/usr/local/bin` directory. To make that happen, you need to run:

```sh
brew link --force libpq
```
{: pre}

Which will symlink (a file that points to another file or folder) all the tools, not just `libpq`, into the `/usr/local/bin` directory.

### Installing `postgresql-client` on Ubuntu
{: #installing-psql-ubuntu}

Linux systems, unlike macOS, have a package manager built in. For Ubuntu (and Debian-based distributions) thats's the `apt` command. The PostgreSQL client is distributed in the appositely named `postgresql-client`. To install it, run a command like:

```sh
sudo apt-get install postgresql-client
```
{: pre}

This will install the PostgreSQL client.

### Installing `postgresql-client` on Red Hat Enterprise Linux
{: #installing-psql-rh-linux}

For Red Hat Enterprise Linux (or RHEL as it's usually written), there's a little more setup than with Ubuntu. For RHEL, the package manager is [`Yum`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum#doc-wrapper){: external}. 

First, you need to point `Yum` to the PostgreSQL repository, like this:

```sh
sudo yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-redhat10-10-2.noarch.rpm
```
{: pre}

`Yum` goes to that URL and configures itself to use that package repository. With that done, you can add packages by name:

```sh
sudo yum install postgresql15
```
{: pre}

This command installs just the client packages. If you are wondering where to find that repository URL, head to Linux Downloads (Red Hat Family) where you'll find a form that will let you select the PostgreSQL version, platform and architecture and it'll give you the appropriate instructions for that Red Hat variant - that includes CentOS, Scientific Linux, and Oracle Enterprise Linux. It also includes Fedora. Fedora's default repositories already have a PostgreSQL client available from them. So For Fedora 27 and 28 and later, install the PostgreSQL client from the terminal with:

```sh
sudo dnf install postgresql.x86_64
```
{: pre}

### Installing `psql` on Windows
{: #installing-psql-windows}

For Windows, use the [PostgreSQL installer from Enterprise DB](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads){: external}. It's a full installation package for PostgreSQL on Windows but you can set it to only install the command line tools like `psql`. Select your PostgreSQL and Windows versions. Once the executable file is downloaded, run it. Select only the *Command Line Tools*, if you don't need the server installed. 

After it installs, you set up your Windows environment variables so that you can use the `psql` client in the command prompt. Go to the **Control panel** > **System and security** > **System** and select* Advanced system settings*. From there you see a box called **System properties**. Select **Environment variables**. A window appears with the two sets of environment variables. In the top set, marked "User variables for...", select the `PATH` entry and then click the **Edit** button. An edit window will appear. Click *New* and add the path to the `psql` client. Your path will depend on where PostgreSQL installed, but typically that would be:

```sh
C:\Program Files\PostgreSQL\<POSTGRES_VERSION>\bin
```
{: pre}

After that, click **OK** a couple of times to go back to the desktop. Start a new Command Prompt and you should be able to run `psql`.

## `psql` Connection Strings
{: #psql-connection-strings}

Connection strings are displayed in the _Endpoints_ panel of your deployment's _Overview_, and can also be retrieved from the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#deployment-connections), and the [API](/apidocs/cloud-databases-api/cloud-databases-api-v5#getconnection).

The information that you need to make a connection with `psql` is in the "cli" section of your connection strings. The table contains a breakdown for reference.

| Field Name|Index|Description|
| ---------- | ----- | ----------- |
| `Bin` | | The recommended binary to create a connection; in this case it is `psql`. |
| `Composed` | | A formatted command to establish a connection to your deployment. The command combines the `Bin` executable, `Environment` variable settings, and uses `Arguments` as command-line parameters. |
| `Environment` | | A list of key/values you set as environment variables. |
| `Arguments` | 0... | The information that is passed as arguments to the command shown in the Bin field. |
| `Certificate` | Base64 | A service proprietary certificate that is used to confirm that an application is connecting to the appropriate server. It is base64 encoded. |
| `Certificate` | Name | The allocated name for the service proprietary certificate. |
| `Type` | | The type of package that uses this connection information; in this case `cli`. |
{: caption="psql/cli connection information" caption-side="top"}

* `0...` indicates that there might be one or more of these entries in an array.

## Creating a command-line client connection
{: #create-cli-connection}

Before creating a command-line client connection, ensure that you have the username and password for your deployment.

The `ibmcloud cdb deployment-connections` command handles everything that is involved in creating a command-line client connection. For example, to connect to a deployment named "example-postgres", use the following command:

```sh
ibmcloud cdb deployment-connections <INSTANCE_NAME_OR_CRN> --start
```
{: pre}

or
```sh
ibmcloud cdb cxn <INSTANCE_NAME_OR_CRN> -s
```
{: pre}

The command prompts for the user password and then runs the `psql` command-line client to connect to the database.

If you have not installed the {{site.data.keyword.databases-for}} CLI plug-in, connect to your PostgreSQL databases using `psql` by giving it the "composed" connection string. It provides environment variables `PGPASSWORD` and `PGSSLROOTCERT`. Set `PGPASSWORD` to the admin's password and `PGSSLROOTCERT` to the path or file name for the service proprietary certificate. 

```sh
PGPASSWORD=$PASSWORD PGSSLROOTCERT=0b22f14b-7ba2-11e8-b8e9-568642342d40 psql 'host=4a8148fa-3806-4f9c-b3fc-6467f11b13bd.8f7bfd7f3faa4218aec56e069eb46187.databases.appdomain.cloud port=32325 dbname=ibmclouddb user=admin sslmode=verify-full'
```
{: .codeblock}

## Using the service proprietary certificate
{: #using-certificate}

1. Copy the certificate information from the _Endpoints_ panel or the Base64 field of the connection information. 
2. If needed, decode the Base64 string into text. 
3. Save the certificate to a file. (You can use the Name that is provided or your own file name).
4. Provide the path to the certificate to the `ROOTCERT` environment variable.

You can display the decoded certificate for your deployment with the CLI plug-in with the command:

```sh
ibmcloud cdb deployment-cacert <INSTANCE_NAME_OR_CRN>
```
{: pre}

The command decodes the base64 into text. Copy and save the command's output to a file and provide the file's path to the `ROOTCERT` environment variable.

Another option is to add `&sslrootcert=/path/to/cert` to your connection string, for example:

```sh
postgres://$USERNAME:$PASSWORD@6eb96148-90bc-49a0-a5a4-dc2b53334653.btdl8mld0r95fevivv30.databases.appdomain.cloud:32109/ibmclouddb?sslmode=verify-full&sslrootcert=/path/to/cert
```
