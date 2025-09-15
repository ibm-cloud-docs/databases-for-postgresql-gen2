---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"

keywords: admin, superuser, roles, service credentials, postgresql users, postgresql service credentials, connection strings, admin password, new user

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# Managing users, roles, and privileges 
{: #user-management}

PostgreSQL uses a system of roles to manage database permissions. Roles are used to give a single user or a group of users a set of privileges. Determine roles, groups, and privileges across your deployment by using the `psql` command `\du`.

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an `admin` user to access and manage PostgreSQL.

## The `admin` user
{: #user-admin}

When you provision a new deployment in {{site.data.keyword.cloud_notm}}, you are automatically given an `admin` user to access and manage PostgreSQL. Once you [set the admin password](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management&interface=ui#user-management-set-admin-password-ui), use it to connect to your deployment.

When `admin` creates a resource in a database, like a table, `admin` owns that object. Resources that are created by `admin` are not accessible by other users, unless you expressly grant permissions to them.

The biggest difference between the `admin` user and any other users you add to your deployment is the [`pg_monitor`](https://www.postgresql.org/docs/current/default-roles.html){: .external} and [`pg_signal_backend`](https://www.postgresql.org/docs/current/default-roles.html){: .external} roles. The `pg_monitor` role provides a set of permissions that makes the admin user appropriate for monitoring the database server. The `pg_signal_backend` role provides the admin user the ability to send signals to cancel queries and connections that are initiated by other users. It is not able to send signals to processes owned by superusers.

You can also use the `admin` user to grant these two roles to other users on your deployment.

To expose the ability to cancel queries to other database users, grant the `pg_signal_backend` role from the `admin` user. Use a command like:

```sql
GRANT pg_signal_backend TO joe;
```
{: .pre}

To allow the user `joe` to cancel backends, grant `pg_signal_backend` to all the users with the `ibm-cloud-base-user` role with a command like:

```sql
GRANT pg_signal_backend TO "ibm-cloud-base-user";
```
{: .pre}

This privilege allows the user or users to terminate any connections to the database.
{: important}

To set up a specific monitoring user, `mary`, use a command like:

```sql
GRANT pg_monitor TO mary;
```
{: .pre}

Grant `pg_signal_backend` to all the users with the `ibm-cloud-base-user` role with a command like:

```sql
GRANT pg_monitor TO "ibm-cloud-base-user";
```
{: .pre}

### Setting the admin password in the UI
{: #user-management-set-admin-password-ui}
{: ui}

Set your admin password through the UI by selecting your instance from the Resource list in the [{{site.data.keyword.cloud_notm}} Dashboard](https://cloud.ibm.com/){: external}. Then, select **Settings**. Next, in the *Change database admin password* section type a new password and click on *Change password*.

### Setting the admin password in the CLI
{: #user-management-set-admin-password-cli}
{: cli}

Use the `cdb user-password` command from the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin) to set the `admin` password.

For example, to set the admin password for a deployment named `example-deployment`, use the following command:

```sh
ibmcloud cdb user-password example-deployment admin <NEW_PASSWORD>
```
{: pre}

### Setting the admin password in the API
{: #user-management-set-admin-password-api}
{: api}

The foundation endpoint that is shown on the Overview panel deployment details section of your service provides the base URL to access this deployment through the API. Use it with the [Set specified user's password](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#changeuserpassword){: external} endpoint to set the admin password.

```sh
curl -X PATCH `https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/users/admin` \
-H `Authorization: Bearer <>` \
-H `Content-Type: application/json` \ 
-d `{"password":"newrootpasswordsupersecure21"}` \
```
{: pre}

## _Service credential_ users
{: #user-management-service-cred}

Users that you [create through the _Service credentials_ panel](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#creating-users-in-_service-credentials_) are members of `ibm-cloud-base-user`. They are able to log in, create users, and create databases.

When a user in a group creates a resource in a database, like a table, all users that are in the same group have access to that resource. Resources that are created by any of the users in `ibm-cloud-base-user` are accessible to other users in `ibm-cloud-base-user`, including the admin user.

## Users created through the CLI
{: #user-management-cli}
{: cli}

Users that you create through the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin) are also members of `ibm-cloud-base-user`. They are able to log in, create users, and create databases.

When a user creates a resource in a database, like a table, all users that are in the same group have access to that resource. Resources that are created by any of the users in `ibm-cloud-base-user` are accessible to other users in `ibm-cloud-base-user`, including the `admin` user.

Users that are created directly from the API and CLI do not appear in _Service Credentials_, but you can [add them](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#adding-users-to-_service-credentials_) if you choose.

## Users created through the API
{: #user-management-api}
{: api}

Users that you create through the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction) are also members of `ibm-cloud-base-user`. They are able to log in, create users, and create databases.

When a user creates a resource in a database, like a table, all users that are in the same group have access to that resource. Resources that are created by any of the users in `ibm-cloud-base-user` are accessible to other users in `ibm-cloud-base-user`, including the admin user.

Users that are created directly from the API and CLI do not appear in _Service Credentials_, but you can [add them](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#adding-users-to-_service-credentials_) if you choose.

## The read-only user
{: #user-management-read-only-user}

The `ibm-cloud-base-user-ro` manages privileges for users that are created to access read-only replicas. For more information, see [Configuring read-only replicas](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas).

## The `repl` user
{: #user-management-repl-user}

The `repl` user has Replication privileges and is used if you enable the [`wal2json` plug-in](/docs/databases-for-postgresql?topic=databases-for-postgresql-wal2json) on your deployment. While enabling `wal2json`, set the `repl` user's password, which allows the `wal2json` plug-in to use it.

## Other `ibm` Users
{: #user-management-ibm-users}

If you run the `\du` command with your `admin` account, you might notice users that are named `ibm`, `ibm-cloud-base-user`, `ibm-replication`, and `ibm-rewind`.

The `ibm-cloud-base-user` is used as a template to manage group roles for other users. It is used to manage the users who are created through the CLI and API as well as enable the integration with the _Service Credentials_ user creation on IBM Cloud. A user that is a member of `ibm-cloud-base-user` inherits the create role and create database attributes from `ibm-cloud-base-user`. The `ibm-cloud-base-user` is not able to log in.

The `ibm` account is the only superuser on your deployment. A superuser account is not available for you to us. This user is an internal administrative account that manages the stability of your deployment.

## Users created with `psql`
{: #user-management-psql}

You can bypass creating users through IBM Cloud entirely, and create users directly in PostgreSQL with `psql`. This allows you to use PostgreSQL's native [role and user management](https://www.postgresql.org/docs/current/database-roles.html){: .external}. Users/roles created in `psql` must have all of their privileges set manually, as well as privileges to the objects that they create.

Users that are created directly in PostgreSQL do not appear in _Service credentials_, but you can [add them](/docs/databases-for-postgresql?topic=databases-for-postgresql-connection-strings#adding-users-to-_service-credentials_) if you choose. 

Note that these users are not integrated with IAM controls, even if added to _Service credentials_.
{: .tip}

## Additional users and connection strings
{: #creating_users}

Access to your {{site.data.keyword.databases-for-postgresql}} deployment is not limited to the `admin` user. Add users in the UI in _Service credentials_, with the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin), or the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction).

All users on your deployment can use the connection strings, including connection strings for either public or private endpoints.

When you create a user, it is assigned certain database roles and privileges. These privileges include the ability to log in, create databases, and create other users.

### Creating users in the UI
{: #user-management-creating-users-service-cred}
{: ui}

1. Go to the service dashboard for your service.
2. Click **Service credentials** to open **Service credentials**.
3. Click **New credential**.
4. Choose a descriptive name for your new credential.
5. **Optional** Specify if the new credentials should use a public or private endpoint. Use either `{ "service-endpoints": "public" }` / `{ "service-endpoints": "private" }` in the *Add inline configuration parameters* field to generate connection strings using the specified endpoint. Use of the endpoint is not enforced, it just controls which hostnames are in the connection strings. Private endpoints are generated by default.
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
