---

copyright:
  years: 2025
lastupdated: "2025-11-05"

keywords: manager, superuser, roles, service credentials, postgresql users, postgresql service credentials, connection strings, manager password, new user, Gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Managing users, roles, and privileges 
{: #user-management}

As part of provisioning a new deployment in {{site.data.keyword.cloud}}, you can use the service credential console page to create a user with different roles (Manager and Writer). 

{{site.data.keyword.databases-for-postgresql}} deployments no longer include a default `admin` user. Instead, customers create a user with the `Manager` or `Writer` role using the {{site.data.keyword.cloud}} service credential interface — via UI or CLI. These users come with necessary credentials to connect to and manage the deployment.

### The manager user
{: #user-manager}

The `manager` user functions as a admin-like user and is automatically granted the PostgreSQL default role `pg_monitor`, which provides access to monitoring views and functions within the database. The created user has the CREATEROLE and CREATEDB privileges, inheriting permissions from both `ibm_admin` and `ibm_writer`, enabling broader access and management capabilities within the deployment.

The `manager`user (admin-like) comes with the following roles:

```sh
pg_read_all_data
pg_write_all_data
pg_monitor
pg_read_all_settings
pg_read_all_stats
pg_stat_scan_tables
pg_signal_backend
pg_checkpoint
pg_create_subscription
```
{: pre}

When the `manager` user (admin-like) creates a resource in a database, like a table, the user owns that object. Resources that are created by `manager` user are not accessible by other users, unless you expressly grant permissions to them.

The biggest difference between the `manager` user and any other users you add to your deployment is the [`pg_monitor`](https://www.postgresql.org/docs/current/default-roles.html){: .external} and [`pg_signal_backend`](https://www.postgresql.org/docs/current/default-roles.html){: .external} roles. The `pg_monitor` role provides a set of permissions that makes the `manager` user appropriate for monitoring the database server. The `pg_signal_backend` role provides the `manager` user the ability to send signals to cancel queries and connections that are initiated by other users. It is not able to send signals to processes owned by superusers.

You can also use the `manager` user to grant the following two roles to other users on your deployment.

To expose the ability to cancel queries to other database users, grant the pg_signal_backend role from the `manager` user. Use a command like:

```sql
GRANT pg_signal_backend TO joe;
```
{: .pre}

To set up a specific monitoring user, `mary`, use a command like:

```sql
GRANT pg_monitor TO mary;
```
{: .pre}


### Changing the user password in the UI
{: #user-management-set-manager-password-ui}
{: ui}

Changing a user password is not supported via the {{site.data.keyword.cloud_notm}} console on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```sh
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

### Creating the manager user in the CLI
{: #user-management-create-manager-user-cli}
{: cli}

Use one of the following commands from the {{site.data.keyword.cloud_notm}} CLI {{site.data.keyword.databases-for}} plug-in to create the `manager` user.

```sh
ibmcloud resource service-key-create <service_key_name> Manager --instance-name <instance_name>
{: pre}

```sh
ibmcloud resource service-key-create <service_key_name> Manager --instance-id <guid>
```
{: pre}

These commands can be used when creating a user with either the Writer or Manager role. In this example, the command creates a PostgreSQL user (ibmcloud_d0388…) with CREATEROLE and CREATEDB privileges. This user inherits permissions from both `ibm_admin` and `ibm_writer`, enabling broader access and management capabilities within the deployment.

```sh
|                  rolname                  | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin |                                                                             memberof                                   |
|-------------------------------------------|----------|------------|---------------|-------------|-------------|------------------------------------------------------------------------------------------------------------------------|
| ibm_admin                                 | f        | t          | f             | f           | f           | {pg_read_all_data,pg_write_all_data,pg_monitor,pg_read_all_settings,pg_read_all_stats,pg_stat_scan_tables,pg_signal_backend,pg_checkpoint,pg_create_subscription} |
| ibm_monitoring                            | f        | f          | f             | f           | t           | {pg_monitor,pg_use_reserved_connections} |
| ibm_replication                           | f        | t          | f             | f           | t           | {pg_use_reserved_connections} |
| ibm_rest                                  | f        | f          | t             | t           | t           | {pg_use_reserved_connections,ibm_admin,ibm_writer,ibmcloud_d0388c96d13841b1b45f1039c73ea6c7} |
| ibm_rewind                                | f        | t          | f             | f           | t           | {} |
| ibm_superuser                             | t        | t          | t             | t           | t           | {} |
| ibm_writer                                | f        | t          | f             | f           | f           | {pg_read_all_data,pg_write_all_data} |
| ibmcloud_d0388c96d13841b1b45f1039c73ea6c7 | f        | t          | t             | t           | t           | {ibm_admin,ibm_writer} |
```
{: pre}

Similarly, for creating a user with the `Writer` role, use the following command:

```sh
ibmcloud resource service-key-create <service_key_name> Writer --instance-name <instance_name>
```
{: pre}

The user (`ibmcloud_b153...`) with `writer` role inherits the `ibm_writer` permissions:

```sh
|                 rolname                  | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin |                                                                             memberof                                   |
|------------------------------------------|----------|------------|---------------|-------------|-------------|------------------------------------------------------------------------------------------------------------------------|
| ibm_admin                                 | f        | t          | f             | f           | f           | {pg_read_all_data,pg_write_all_data,pg_monitor,pg_read_all_settings,pg_read_all_stats,pg_stat_scan_tables,pg_signal_backend,pg_checkpoint,pg_create_subscription}|
| ibm_monitoring                            | f        | f          | f             | f           | t           | {pg_monitor,pg_use_reserved_connections} |
| ibm_replication                           | f        | t          | f             | f           | t           | {pg_use_reserved_connections} |
| ibm_rest                                  | f        | f          | t             | t           | t           | {pg_use_reserved_connections,ibm_admin,ibm_writer,ibmcloud_b153b496ae5a41a08a27db730e43b835} |
| ibm_rewind                                | f        | t          | f             | f           | t           | {} |
| ibm_superuser                             | t        | t          | t             | t           | t           | {} |
| ibm_writer                                | f        | t          | f             | f           | f           | {pg_read_all_data,pg_write_all_data} |
| ibmcloud_b153b496ae5a41a08a27db730e43b835 | f        | t          | f             | f           | t           | {ibm_writer} |
```
{: pre}

### Deleting the user in the CLI
{: #user-management-delete-manager-user-cli}
{: cli}

Use the following command from the {{site.data.keyword.cloud_notm}} CLI {{site.data.keyword.databases-for}} plug-in to delete the created user.

```sh
ibmcloud resource service-key-delete <service_key_name> 
```
{: pre}

### Changing the user password in the CLI
{: #user-management-set-manager-pw-cli}
{: cli}

Changing a user password is not supported via the CLI on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```sh
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

### Changing the user password through the API
{: #user-management-set-manager-password-api}
{: api}

Changing a user password is not supported via API on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```sh
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

## Users created with `psql`
{: #user-management-psql}

You can bypass creating users through IBM Cloud entirely, and create users directly in PostgreSQL with `psql`. This allows you to use PostgreSQL's native [role and user management](https://www.postgresql.org/docs/current/database-roles.html){: .external}. Users/roles created in `psql` must have all of their privileges set manually, as well as privileges to the objects that they create.

Users that are created directly in PostgreSQL do not appear in _Service credentials_, but you can [add them](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#adding-users-to-_service-credentials_) if you choose. 

Note that these users are not integrated with IAM controls, even if added to _Service credentials_.
{: .tip}

## Additional users and connection strings
{: #creating_users}

Access to your {{site.data.keyword.databases-for-postgresql}} deployment is not limited to the `manager` user. Additional users can be created using the CLI, with the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin), or the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction).

All users on your deployment can use the connection strings, including connection strings for private endpoints.

When you create a user, it is assigned certain database roles and privileges. These privileges include the ability to log in, create databases, and create other users.

### Creating users in the UI
{: #user-management-creating-users-service-cred}
{: ui}

1. Go to the service dashboard for your service.
2. Click **Service credentials** to open **Service credentials**.
3. Click **New credential**.
4. Choose a descriptive name for your new credential.
5. **Optional** Specify if the new credentials should use a public or private endpoint. Use `{ "service-endpoints": "private" }` in the *Add inline configuration parameters* field to generate connection strings using the specified endpoint. Use of the endpoint is not enforced, it just controls which hostnames are in the connection strings. Private endpoints are generated by default. On Gen 2, only `private` endpoints are supported.
6. Click **Add** to provision the new credentials. A username and password, and an associated database user in the PostgreSQL database are auto-generated.

The new credentials appear in the table, and the connection strings are available as JSON in a click-to-copy field under _View credentials_.

### Creating users from the CLI
{: #user-management-creating-users-cli}
{: cli}

If you manage your service through the {{site.data.keyword.cloud_notm}} CLI and the [cloud databases plug-in](/docs/cli?topic=cli-install-ibmcloud-cli), you can create a new user with `cdb user-create`. For example, to create a new user for an "example-deployment", use the following command:

```sh
ibmcloud cdb user-create example-deployment <NEW_USER_NAME> <NEW_PASSWORD>
```
{: pre}

Once the task has finished, you can retrieve the new user's connection strings with the `ibmcloud cdb deployment-connections` command.

### Creating users from the API
{: #user-management-creating-users-api}
{: api}

The _Foundation endpoint_ that is shown on the _Overview_ panel _Deployment details_ of your service provides the base URL to access this deployment through the API. To create and manage users, use the base URL with the `/users` endpoint.

```sh
curl -X POST 'https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/users' \
-H "Authorization: Bearer $APIKEY" \
-H "Content-Type: application/json" \
-d '{"username":"jane_smith", "password":"newsupersecurepassword"}'
```
{: pre}

After the task finishes, retrieve the new user's connection strings from the `/users/{userid}/connections` endpoint.

### Adding users to _Service credentials_
{: #user-management-adding-users-service-cred}

Creating a new user from the CLI or API doesn't automatically populate that user's connection strings into _Service credentials_. To add them, create a new credential with the existing user information.

Enter the username and password in the JSON field _Add inline configuration parameters_, or specify a file where the JSON information is stored. For example, putting `{"existing_credentials":{"username":"Robert","password":"supersecure"}}` in the field generates _Service credentials_ with the username "Robert" and password "supersecure" filled into connection strings.

Generating credentials from an existing user does not check for or create that user.
{: tip}
