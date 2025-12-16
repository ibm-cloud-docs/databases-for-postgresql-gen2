---

copyright:
  years: 2025
lastupdated: "2025-12-16"

keywords: pgAdmin, postgresql gui, PostgreSQL, PostgreSQL cloud database, PostgreSQL getting started, Gen 2

subcollection: databases-for-postgresql-gen2

content-type: tutorial
services:
account-plan: paid
completion-time: 30m

---

{{site.data.keyword.attribute-definition-list}}

# Getting started with {{site.data.keyword.databases-for-postgresql}}
{: #getting-started}
{: toc-content-type="tutorial"}
{: toc-services=""}
{: toc-completion-time="30m"}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

This tutorial guides you through the steps to quickly start by using {{site.data.keyword.databases-for-postgresql}} on the Gen 2 platform by provisioning an instance, setting up a secure connection through a VSI and VPE, and enabling logging and monitoring.

Follow these steps to complete the tutorial:
{: ui}

* [Before you begin](#prereqs)
* [Step 1: Provision through the console](#provision_instance_ui)
* [Step 2: Creating the Manager user through the console](#manager_user)
* [Step 3: Users created with psql](#psql)
* [Step 4: Set up context-based restrictions](#postgresql_cbr)
* [Step 5: Create a connection](#connect_setup)
* [Step 6: Connect {{site.data.keyword.mon_full_notm}}](#connect_monitoring_ui)
* [Step 7: Connect IBM Cloud Logs](#postgresql_logs)
* [Next Steps](#next_steps)
{: ui}

Follow these steps to complete the tutorial:
{: cli}

* [Before you begin](#prereqs)
* [Step 1: Provision through the CLI](#provision_instance_cli)
* [Step 2: Creating the Manager user through the console](#manager_user)
* [Step 3: Users created with psql](#psql)
* [Step 4: Set up context-based restrictions](#postgresql_cbr)
* [Step 5: Create a connection](#connect_setup)
* [Step 6: Connect {{site.data.keyword.mon_full_notm}}](#connect_monitoring_ui)
* [Step 7: Connect {{site.data.keyword.atracker_full}}](#activity_tracker_ui)
* [Next Steps](#next_steps)
{: cli}

Follow these steps to complete the tutorial:
{: api}

* [Before you begin](#prereqs)
* [Step 1: Provision through the API](#provision_instance_api)
* [Step 2: Creating the Manager user through the console](#manager_user)
* [Step 3: Users created with psql](#psql)
* [Step 4: Set up context-based restrictions](#postgresql_cbr)
* [Step 5: Create a connection](#connect_setup)
* [Step 6: Connect {{site.data.keyword.mon_full_notm}}](#connect_monitoring_ui)
* [Step 7: Connect {{site.data.keyword.atracker_full}}](#activity_tracker_ui)
* [Next Steps](#next_steps)
{: api}

Follow these steps to complete the tutorial:
{: terraform}

* [Before you begin](#tf_prereqs)
* [Step 1: Setup](#tf_setup)
* [Step 2: Configure IBM Cloud Provider](#tf_configure_provider)
* [Step 3: Project Structure](#tf_project_structure)
* [Step 4: Configuration Files](#tf_configuration_files)
* [Step 5: Deployment](#tf_deployment)
* [Step 6: Post-Deployment](#tf_post_deployment)
* [Step 7: Connect To Your Database](#tf_connect_database)
{: terraform}

## Before you begin
{: #prereqs}

* You need an [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){: external}.

## Step 1: Provision through the console
{: #provision_instance_ui}
{: ui}

1. Log in to the {{site.data.keyword.cloud_notm}} console.
1. Click the [**{{site.data.keyword.databases-for-postgresql}} service**](https://cloud.ibm.com/catalog){: external} in the **catalog**.

1. In **Service details**, configure the following:
    - **Location** - Select a location that supports Gen 2
    - **Service name** - The name can be any string and is the name that is used on the web and in the CLI to identify the new instance.
    - **Resource group** - If you are organizing your services into resource groups. For more information, see [Managing resource groups](/docs/account?topic=account-rgs).

1. **Resource allocation** - Select an isolated compute instance with a certain amount of RAM and CPU cores. Changing resource allocation requires selecting a different host size. *Once provisioned, disk cannot be scaled down.*
1. In **Service configuration**, configure the following:
    - **Database version** [Set only at deployment]{: tag-red} - The deployment version of your database. To ensure optimal performance, run the preferred version. The latest minor version is used automatically. For more information, see [Database versioning policy](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-versioning-policy&interface=ui){: external}.
    - **Encryption** - If you use [Key Protect](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-key-protect&interface=ui), an instance and key can be selected to encrypt the instance's disk. If you do not use your own key, the instance automatically creates and manages its own disk encryption key.

1. Click **Create**. The {{site.data.keyword.databases-for}} **Resource list** page opens.

1. When your instance has been provisioned, click the instance name to view more information.

As part of provisioning a new instance in {{site.data.keyword.cloud}}, you can use the service credential console page to create a user with different roles (Manager and Writer).
{: note}

{{site.data.keyword.databases-for-postgresql}} instances no longer include a default `admin` user. Instead, customers create a user with the `Manager` or `Writer` role using the {{site.data.keyword.cloud}} service credential interface — via UI or CLI. These users come with necessary credentials to connect to and manage the instance.

## Step 1: Provision through the CLI
{: #provision_instance_cli}
{: cli}

You can provision a {{site.data.keyword.databases-for-postgresql}} instance through the CLI. If you don't already have it, you need to install the [{{site.data.keyword.cloud_notm}} CLI](https://www.ibm.com/cloud/cli){: external}.

1. Log in to {{site.data.keyword.cloud_notm}} with the following command:
{: #step2_login_qsg}

    ```sh
    ibmcloud login
    ```
    {: pre}

    If you use a federated user ID, it's important that you switch to a one-time passcode (`ibmcloud login --sso`), or use an API key (`ibmcloud --apikey key` or `@key_file`) to authenticate. For more information about how to log in using the CLI, see [General CLI (ibmcloud) commands](/docs/cli?topic=cli-ibmcloud_cli#ibmcloud_login) under `ibmcloud login`.

1. Create a {{site.data.keyword.databases-for-postgresql}} instance.
{: #step3_es_instance}

    Select one of the following methods:

    * To create an instance from the CLI on the Enterprise plan, run the following command:

        ```sh
        ibmcloud resource service-instance-create <INSTANCE_NAME> databases-for-postgresql standard-gen2 <LOCATION> -g <RESOURCE_GROUP>
        ```
        {: codeblock}

      This provisions a PG instance with the default of 2 members and 10 GB disk, as well as the smallest host flavor available in the location you selected.

      Alternatively, you can specify custom values for these parameters by passing in the parameters flag:

      ```sh
      ibmcloud resource service-instance-create <INSTANCE_NAME> databases-for-postgresql standard-gen2 <LOCATION> -g <RESOURCE_GROUP> -p '{"dataservices": {"postgresql": {"storage_gb": 40, "members": 3, "host_flavor": "bx3d.8x40"}}}'
      ```
      {: codeblock}

      This will provision a PG instance with 3 members, 40GB of storage per member running on hosts of flavor bx3d.8x40.

      If you pass in unsupported values, the `create` command will fail with a message indicating which values are invalid, for example, when passing in an unsupported number of members:

      ```Message: The broker for 'Databases for PostgreSQL' service returned error, [400, Bad Request] invalid members '5', allowed values: 2, 3. If this is unexpected, open a support ticket with the service to help troubleshoot the issue.```

      Or, an unsupported disk size:

      ```Message: The broker for 'Databases for PostgreSQL' service returned error, [400, Bad Request] invalid storage size '10000', only integer values between 10 and 9600 are allowed. If this is unexpected, open a support ticket with the service to help troubleshoot the issue.```

      Passing in any other key-value pairs that are not supported by the service will result in a corresponding error message as well.

      Supported parameters:

      - storage_gb (valid integer values between 10 and 9600, representing disk storage per member in GB)
      - members(valid integer values '2' and '3', indicating whether to run PG with 2 zone HA or 3 zone HA).
      - host_flavor(values depend on location)

   The fields in the command are described in the table that follows.

   | Field | Description | Flag |
   |-------|------------|------------|
   | `NAME` [Required]{: tag-red} | The instance name can be any string and is the name that is used on the web and in the CLI to identify the new instance. |  |
   | `SERVICE_NAME` [Required]{: tag-red} | Name or ID of the service. For {{site.data.keyword.databases-for-postgresql}}, use `databases-for-postgresql`. |  |
   | `SERVICE_PLAN_NAME` [Required]{: tag-red} | Standard plan (`standard`). |  |
   | `LOCATION` [Required]{: tag-red} | The location where you want to deploy. To retrieve a list of regions, use the `ibmcloud regions` command. |  |
   | `SERVICE_ENDPOINTS_TYPE` | Configure the [Service endpoints](/docs/cloud-databases?topic=cloud-databases-service-endpoints) of your instance, `private`. |  |
   | `RESOURCE_GROUP` | The Resource group name. The default value is `default`. | -g |
   | `--parameters` | JSON file or JSON string of parameters to create service instance. | -p |
   {: caption="Basic command format fields" caption-side="top"}

   You see a response like:

   ```text
   Service instance <INSTANCE_NAME> was created.

Name:                   <INSTANCE_NAME>
ID:                     crn:v1:bluemix:public:databases-for-postgresql:<LOCATION>:a/<ACCOUNT>:<INSTANCE_GUID>::
GUID:                   <INSTANCE_GUID>
Location:               <LOCATION>
State:                  provisioning
Type:                   service_instance
Sub Type:               Public
Allow Cleanup:          false
Locked:                 false
One-time credentials:   false
Created at:             <CREATED_AT_TIMESTAMP>
Updated at:             <UPDATED_AT_TIMESTAMP>
Last Operation:
                        Status    create in progress
                        Message   Started create instance operation
   ```
   {: codeblock}

1. To check provisioning status, use the following command:

   ```sh
   ibmcloud resource service-instance <INSTANCE_NAME>
   ```
   {: pre}

   When complete, you see a response like:

   ```Retrieving service instance <INSTANCE_NAME> in resource group default under account <ACCOUNT> as <USER_ID>...
OK
Name:                   <INSTANCE_NAME>
ID:                     crn:v1:bluemix:public:databases-for-postgresql:<LOCATION>:a/<ACCOUNT>:<INSTANCE_GUID>::
GUID:                   <INSTANCE_GUID>
Location:               LOCATION
Service Name:           databases-for-postgresql
Service Plan Name:      standard-gen2
Resource Group Name:    Default
State:                  active
Type:                   service_instance
Sub Type:               Public
Locked:                 false
One-time credentials:   false
Created at:             <CREATED_AT_TIMESTAMP>
Created by:             <USER_ID>
Updated at:             <UPDATED_AT_TIMESTAMP>
Last Operation:
                        Status    create succeeded
                        Message   Provision completed successfully
   ```
   {: codeblock}

1. (Optional) Deleting a service instance
   Delete an instance by running a command like this one:

   ```sh
   ibmcloud resource service-instance-delete <INSTANCE_NAME>
   ```
   {: pre}

### Connect to your database with the CLI
{: #connecting-cli}

Find the appropriate commands to connect to your database from the CLI in [Cloud Databases CLI Reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference) and [Connecting with psql](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-connecting-psql).

The `ibmcloud cdb deployment-connections` command handles everything that is involved in creating a CLI connection. For example, to connect to an instance named "example-postgres", use a command like:

```sh
ibmcloud cdb deployment-connections example-postgres --start
```
{: pre}

The command prompts for the `manager` user password and then runs the `psql` CLI to connect to the database. To install the Cloud Databases plug-in, see [Connecting with psql documentation here](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-connecting-psql).

### The `--parameters` parameter
{: #flags-params-service-endpoints}
{: cli}

The `service-instance-create` command supports a `-p` flag, which allows JSON-formatted parameters to be passed to the provisioning process. Some parameter values are Cloud Resource Names (CRNs), which uniquely identify a resource in the cloud. All parameter names and values are passed as strings.

For example, if a database is being provisioned from a particular backup and the new database instance needs a total of 9 GB of memory across three members, then the command to provision 3 GBs per member looks like:

```sh
ibmcloud resource service-instance-create databases-for-postgresql <SERVICE_NAME> standard us-south \
-p \ '{
  "backup_id": "crn:v1:blue:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71717:backup:06392e97-df90-46d8-98e8-cb67e9e0a8e6",
  "members_memory_allocation_mb": "3072"
}'
```
{: .pre}

## Step 1: Provision through the resource controller API
{: #provision_instance_api}
{: api}

Follow these steps to provision by using the [resource controller API](https://cloud.ibm.com/apidocs/resource-controller/resource-controller){: external}.

1. Obtain an [IAM token from your API token](https://cloud.ibm.com/apidocs/resource-controller/resource-controller#authentication){: external}.
1. You need to know the ID of the resource group to which you would like to deploy. This information is available through the [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-ibmcloud_commands_resource#ibmcloud_resource_groups).

   Use a command like:

   ```sh
   ibmcloud resource groups
   ```
   {: pre}

1. You need to know the region where you would like to deploy.

   To list all of the regions that instances can be provisioned into from the current region, use the [{{site.data.keyword.databases-for}} CLI plug-in](https://cloud.ibm.com/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference){: external}.

   The command looks like:

   ```sh
   ibmcloud cdb regions --json
   ```
   {: pre}


   Once you have all the information, [provision a new resource instance](https://cloud.ibm.com/apidocs/resource-controller/resource-controller#create-resource-instance){: external} with the {{site.data.keyword.cloud_notm}} resource controller.

   ```sh
   curl -X POST \
     https://resource-controller.cloud.ibm.com/v2/resource_instances \
     -H 'Authorization: Bearer <>' \
     -H 'Content-Type: application/json' \
       -d '{
       "name": "my-instance",
       "target": "blue-us-south",
       "resource_group": "5g9f447903254bb58972a2f3f5a4c711",
       "resource_plan_id": "databases-for-postgresql-standard"
     }'
   ```
   {: .pre}

   The parameters `name`, `target`, `resource_group`, and `resource_plan_id` are all required.
   {: required}

Supported parameters:

      - `storage_gb` (valid integer values are between 10 and 9600, representing disk storage per member in GB)
      - `members`(valid integer values are '2' and '3', indicating whether to run PG with two-zone HA or three-zone HA)
      - `host_flavor`(values depend on location).

## List of additional parameters
{: #provisioning-parameters-api}
{: api}

* `backup_id`- A CRN of a backup resource to restore from. The backup must be created by a database instance with the same service ID. The backup is loaded after provisioning and the new instance starts up that uses that data. A backup CRN is in the format `crn:v1:<...>:backup:<uuid>`. If omitted, the database is provisioned empty.
* `version` - The version of the database to be provisioned. If omitted, the database is created with the most recent major and minor version.
* `disk_encryption_key_crn` - The CRN of a KMS key (for example, [{{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-get-started) or [{{site.data.keyword.keymanagementserviceshort}}](/docs/key-protect?topic=key-protect-about)), which is then used for disk encryption. A KMS key CRN is in the format `crn:v1:<...>:key:<id>`.
* `backup_encryption_key_crn` - The CRN of a KMS key (for example, [{{site.data.keyword.hscrypto}}](/docs/hs-crypto?topic=hs-crypto-get-started) or [{{site.data.keyword.keymanagementserviceshort}}](/docs/key-protect?topic=key-protect-about)), which is then used for backup encryption. A KMS key CRN is in the format `crn:v1:<...>:key:<id>`.

   To use a key for your backups, you must first [enable the service-to-service delegation](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-key-protect&interface=ui#key-byok).
   {: note}

* `service_endpoints` - The [Service endpoints](/docs/cloud-databases?topic=cloud-databases-service-endpoints) supported on your instance,`private`. This is a required parameter.

### Using APIs
{: #using_apis}
{: api}

Use the [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external} to work with your {{site.data.keyword.databases-for-postgresql}} instance. The resource controller API is used to [provision an instance](#provision_instance_api).

## Step 2: Create the manager (admin-like) user
{: #admin_like}

### The manager user
{: #admin_like_manager_user}

As part of provisioning a new instance in {{site.data.keyword.cloud}}, you can use the service credential console page to create a user with different roles (Manager and Writer).

{{site.data.keyword.databases-for-postgresql}} instances no longer include a default `admin` user. Instead, customers create a user with the `Manager` or `Writer` role using the {{site.data.keyword.cloud}} service credential interface — via UI or CLI. These users come with necessary credentials to connect to and manage the instance.

The `manager` user functions as an admin-like user and is automatically granted the PostgreSQL default role `pg_monitor`, which provides access to monitoring views and functions within the database. The created user has the CREATEROLE and CREATEDB privileges, inheriting permissions from both `ibm_admin` and `ibm_writer`, enabling broader access and management capabilities within the instance.

The `manager`user (admin-like) comes with the following roles:

```
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

When the `Manager` user (admin-like) creates a resource in a database, like a table, the user owns that object. Resources that are created by the `Manager` user are not accessible by other users, unless you explicitly grant permissions to them.

The biggest difference between the `Manager` user and any other users you add to your instance is the [`pg_monitor`](https://www.postgresql.org/docs/current/default-roles.html){: .external} and [`pg_signal_backend`](https://www.postgresql.org/docs/current/default-roles.html){: .external} roles. The `pg_monitor` role provides a set of permissions that makes the `manager` user appropriate for monitoring the database server. The `pg_signal_backend` role provides the `manager` user the ability to send signals to cancel queries and connections that are initiated by other users. It is not able to send signals to processes owned by superusers.

You can also use the `Manager` user to grant roles to other users on your instance.

To expose the ability to cancel queries to other database users, grant the pg_signal_backend role from the `Manager` user. Use a command like:

```
GRANT pg_signal_backend TO joe;
```
{: .pre}

To set up a specific monitoring user, `mary`, use a command like:

```
GRANT pg_monitor TO mary;
```
{: .pre}


### Change the user password in the UI
{: #user-management-set-manager-password-ui}
{: ui}

Changing a user password is not supported via the {{site.data.keyword.cloud_notm}} console on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

### Create the manager user in the CLI
{: #manager_user_set_cli}
{: cli}

Use one of the following commands from the {{site.data.keyword.cloud_notm}} CLI {{site.data.keyword.databases-for}} plug-in to create the `Manager` user.

```
ibmcloud resource service-key-create <service_key_name> --instance-name <INSTANCE_NAME> -p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'
```
{: pre}

```
ibmcloud resource service-key-create <service_key_name> --instance-id <INSTANCE_GUID> -p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager"}'
```
{: pre}

These commands can be used when creating a user with either the Writer or Manager role. In this example, the command creates a PostgreSQL user (ibmcloud_d0388…) with CREATEROLE and CREATEDB privileges. This user inherits permissions from both `ibm_admin` and `ibm_writer`, enabling broader access and management capabilities within the instance.

| rolname | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin | memberof |
|---|---|---|---|---|---|----|
| ibm_admin  | f  | t  | f  | f | f | {pg_read_all_data,pg_write_all_data,pg_monitor,pg_read_all_settings,pg_read_all_stats,pg_stat_scan_tables,pg_signal_backend,pg_checkpoint,pg_create_subscription} |
| ibm_monitoring |f| t  | f  | f | t  | {pg_use_reserved_connections} |
| ibm_rest | f  | f  | t | t | t | {pg_use_reserved_connections,ibm_admin,ibm_writer,ibmcloud_d0388c96d13841b1b45f1039c73ea6c7} |
| ibm_rewind | f  | t  | f | f | t | {} |
| ibm_superuser | t | t | t | t | t | {} |
| ibm_writer | f | t | f | f | f | {pg_read_all_data,pg_write_all_data} |
| ibmcloud_d0388c96d13841b1b45f1039c73ea6c7 | f | t | t | t | t | {ibm_admin,ibm_writer} |
{: caption="`ibm_admin` and `ibm_writer` permissions" caption-side="top"}

Similarly, for creating a user with the `Writer` role, use the following command, updating the 'role_crn' to 'Writer' instead of 'Manager:

```
ibmcloud resource service-key-create <service_key_name> --instance-name <INSTANCE_NAME> -p '{"role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Writer"}'
```
{: pre}

The user (`ibmcloud_b153...`) with `writer` role inherits the `ibm_writer` permissions:

|rolname | rolsuper | rolinherit | rolcreaterole | rolcreatedb | rolcanlogin | memberof |
|---|---|---|---|---|---|---|
| ibm_admin | f | t | f | f | f | {pg_read_all_data,pg_write_all_data,pg_monitor,pg_read_all_settings,pg_read_all_stats,pg_stat_scan_tables,pg_signal_backend,pg_checkpoint,pg_create_subscription}|
| ibm_monitoring | f | f | f | f | t | {pg_monitor,pg_use_reserved_connections} |
| ibm_replication | f | t | f | f | t | {pg_use_reserved_connections} |
| ibm_rest | f | f | t | t | t | {pg_use_reserved_connections,ibm_admin,ibm_writer,ibmcloud_b153b496ae5a41a08a27db730e43b835} |
| ibm_rewind | f | t | f | f | t | {} |
| ibm_superuser | t | t | t | t | t | {} |
| ibm_writer | f | t | f | f | f | {pg_read_all_data,pg_write_all_data} |
| ibmcloud_b153b496ae5a41a08a27db730e43b835 | f | t | f | f | t | {ibm_writer} |
{: caption="`ibm_writer`permissions" caption-side="top"}

### Delete the user in the CLI
{: #manager_user_del_cli}
{: cli}

Use the following command from the {{site.data.keyword.cloud_notm}} CLI {{site.data.keyword.databases-for}} plug-in to delete the created user.

```sh
ibmcloud resource service-key-delete <service_key_name>
```
{: pre}

### Change the manager password in the CLI
{: #manager_pw_set_cli}
{: cli}

Changing a user password is not supported via the CLI on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```sh
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

### Change the manager password through the API
{: #manager_pw_set_api}
{: api}

Changing a user password is not supported via API on Gen 2. However, you can update a password using tools, such as `psql` by executing the following command:

```sh
ALTER ROLE username WITH PASSWORD 'new_password';
```
{: pre}

### Use `psql`
{: #using-pgadmin}

## Users created with `psql`
{: #user-management-psql}

You can bypass creating users through the {{site.data.keyword.cloud_notm}} entirely, and create users directly in PostgreSQL with `psql`. This allows you to use PostgreSQL's native [role and user management](https://www.postgresql.org/docs/current/database-roles.html){: .external}. Users and roles created in `psql` must have all of their privileges set manually, as well as privileges to the objects that they create.

Users that are created directly in PostgreSQL do not appear in _Service credentials_, but you can [add them](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-connection-strings&interface=ui) if you choose.

Note that these users are not integrated with IAM controls, even if added to _Service credentials_.
{: .tip}

## Additional users and connection strings
{: #creating_users}

Access to your {{site.data.keyword.databases-for-postgresql}} instance is not restricted to users with `Manager` privileges only. Any user having appropriate permissions, whether created through the CLI, with the [{{site.data.keyword.databases-for}} CLI plug-in](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-cdb-reference), or via `psql`, can also access the database instance.

All users on your instance can use the connection strings, including connection strings for private endpoints.

When you create a user, it is assigned certain database roles and privileges. These privileges include the ability to log in, create databases, and create other users.


## Step 4: Set up context-based restrictions
{: #postgresql_cbr}

Context-based restrictions give account owners and administrators the ability to define and enforce access restrictions for {{site.data.keyword.cloud}} resources based on the context of access requests. Access to {{site.data.keyword.databases-for}} resources can be controlled with context-based restrictions and Identity and Access Management (IAM) policies.

To set up context-based restrictions for your {{site.data.keyword.databases-for-postgresql}} instance, follow the steps at [Protecting {{site.data.keyword.databases-for}} resources with context-based restrictions](/docs/cloud-databases?topic=cloud-databases-cbr){: external}.

## Step 5: Create a connection
{: #create_connection}

Follow these subtopics to set up your environment and connect to the provisioned database:

* [Create a VPC](https://cloud.ibm.com/infrastructure/network/vpcs/) (Virtual Private Cloud): A VPC is your own isolated network within {{site.data.keyword.cloud}} where you can securely run resources.
* [Generate an SSH key](https://cloud.ibm.com/infrastructure/compute/sshKeys/): SSH keys allow you to securely connect to your virtual servers.
* [Provision a Virtual Server Instance (VSI)](https://cloud.ibm.com/infrastructure/compute/vs/): A VSI is your cloud-based server where applications and workloads will run.
* [Reserve a floating IP for your VSI](https://cloud.ibm.com/infrastructure/network/floatingIPs/): A floating IP is a public IP address that lets you access your VSI from the internet.
* [Create a Virtual Private Endpoint (VPE)](https://cloud.ibm.com/infrastructure/network/endpointGateways/): A VPE provides secure, private connectivity to {{site.data.keyword.cloud_notm}} services.


## Step 6: Connect {{site.data.keyword.mon_full_notm}} through the console
{: #connect_monitoring_ui}
{: ui}

You can use {{site.data.keyword.mon_full_notm}} to get operational visibility into the performance and health of your applications, services, and platforms. {{site.data.keyword.mon_full_notm}} provides administrators, DevOps teams, and developers full stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards.

For more information about how to use {{site.data.keyword.monitoringshort}} with {{site.data.keyword.databases-for-postgresql}}, see [Monitoring integration](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-monitoring&interface=ui).

## Step 6: Connect {{site.data.keyword.mon_full_notm}} through the CLI
{: #connect_monitoring_cli}
{: cli}

You can use {{site.data.keyword.mon_full_notm}} to get operational visibility into the performance and health of your applications, services, and platforms. {{site.data.keyword.mon_full_notm}} provides administrators, DevOps teams, and developers full stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards.

For more information about how to use {{site.data.keyword.monitoringshort}} with {{site.data.keyword.databases-for-postgresql}}, see [Monitoring integration](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-monitoring&interface=ui).

You cannot connect {{site.data.keyword.mon_full_notm}} by using the CLI. Use the console to complete this task. For more information, see [Monitoring integration](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-monitoring&interface=ui).
{: note}

## Step 6: Connect {{site.data.keyword.mon_full_notm}} through the API
{: #connect_monitoring_api}
{: api}

You can use {{site.data.keyword.mon_full_notm}} to get operational visibility into the performance and health of your applications, services, and platforms. {{site.data.keyword.mon_full_notm}} provides administrators, DevOps teams, and developers full stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards.

For more information about how to use {{site.data.keyword.monitoringshort}} with {{site.data.keyword.databases-for-postgresql}}, see [Monitoring integration](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-monitoring&interface=ui).

You cannot connect {{site.data.keyword.mon_full_notm}} by using the CLI. Use the console to complete this task. For more information, see [Monitoring integration](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-monitoring&interface=ui).
{: note}

## Step 7: Connect IBM Cloud Logs (#postgresql_logs)
{: #acloudlogs_ui}
{: ui}


## Step 7: Connect {{site.data.keyword.atracker_full_notm}} through the CLI
{: #activity_tracker_cli}
{: cli}

{{site.data.keyword.atracker_full}} allows you to view, manage, and audit service activity to comply with corporate policies and industry regulations. {{site.data.keyword.atracker_short}} records user-initiated activities that change the state of a service in {{site.data.keyword.cloud_notm}}. Use {{site.data.keyword.atracker_short}} to track how users and applications interact with the {{site.data.keyword.databases-for-postgresql}} service.

To get up and running with {{site.data.keyword.atracker_short}}, see [Getting started with {{site.data.keyword.atracker_short}}](/docs/atracker?topic=atracker-getting-started){: external}.

{{site.data.keyword.atracker_short}} can have only one instance per location. To view events, you must access the web UI of the {{site.data.keyword.atracker_short}} service in the same location where your service instance is available. For more information, see [Launch the web UI](/docs/cloud-logs?topic=cloud-logs-getting-started#gs-access-ui){: external}.

For more information about events specific to {{site.data.keyword.databases-for-postgresql}}, see [Activity tracking events](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-at_events&interface=api).

Events are formatted according to the Cloud Auditing Data Federation (CADF) standard. For further details of the information they include, see [CADF standard](/docs/atracker?topic=atracker-event){: external}.

You cannot connect {{site.data.keyword.atracker_short}} by using the CLI. Use the console to complete this task. For more information, see [Activity tracking events](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-at_events&interface=api).
{: note}

## Step 7: Connect {{site.data.keyword.atracker_full}} through the API
{: #activity_tracker_api}
{: api}

{{site.data.keyword.atracker_full_notm}} allows you to view, manage, and audit service activity to comply with corporate policies and industry regulations. {{site.data.keyword.atracker_short}} records user-initiated activities that change the state of a service in {{site.data.keyword.cloud_notm}}. Use {{site.data.keyword.atracker_short}} to track how users and applications interact with the {{site.data.keyword.databases-for-postgresql}} service.

To get up and running with {{site.data.keyword.atracker_short}}, see [Getting Started with {{site.data.keyword.atracker_short}}]/docs/atracker?topic=atracker-getting-started){: external}.

{{site.data.keyword.atracker_short}} can have only one instance per location. To view events, you must access the web UI of the {{site.data.keyword.atracker_short}} service in the same location where your service instance is available. For more information, see [Launch the web UI](/docs/activity-tracker?topic=activity-tracker-getting-started#gs_step4){: external}.

For more information about events specific to {{site.data.keyword.databases-for-postgresql}}, see [Activity tracking events](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-at_events&interface=api).

Events are formatted according to the Cloud Auditing Data Federation (CADF) standard. For further details of the information they include, see [CADF standard](/docs/atracker?topic=atracker-event){: external}.

You cannot connect {{site.data.keyword.atracker_short}} by using the API. Use the console to complete this task. For more information, see [Activity tracking events](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-at_events&interface=api).
{: note}

## Next steps
{: #next-steps}

- If you are using {{site.data.keyword.databases-for-postgresql}} for the first time, see the [official {{site.data.keyword.databases-for-postgresql}} documentation](https://www.postgresql.org/docs/){: external}.
- Secure your instance by adding [context-based restrictions](/docs/cloud-databases?topic=cloud-databases-cbr&interface=ui).
- Connect your instance to [IBM Cloud Log Analysis](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-logging&interface=ui) and [IBM Cloud Monitoring](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-monitoring&interface=ui) for observability and alerting.
- Connect to and manage your databases and data with {{site.data.keyword.databases-for-postgresql}}'s CLI tool [`psql`](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-connecting-psql).
- Looking for more tools on managing your databases? Connect to your instance with the following tools:
    - [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-install-ibmcloud-cli){: external}
    - [{{site.data.keyword.databases-for}} CLI plug-in](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference){: external}
    - [{{site.data.keyword.databases-for}} API](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-api){: external}
- If you plan to use {{site.data.keyword.databases-for-postgresql}} for your applications, see:
    - [Connecting an external application](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-external-app)
    - [Connecting an {{site.data.keyword.cloud_notm}} application](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-ibmcloud-app)

- To ensure the stability of your applications and your databases, see:
    - [High availability](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-ha-dr)
    - [Performance](/docs/databases-for-postgresql-gen2?topic=databases-for-postgresql-gen2-performance&interface=ui)

## Before you begin
{: #tf_prereqs}
{: terraform}

Terraform is an open-source infrastructure as code (IaC) tool that allows you to define and provision cloud infrastructure using declarative configuration files.
For {{site.data.keyword.databases-for-postgresql}}, Terraform provides a powerful way to manage your database instances programmatically rather than through manual console operations. Using Terraform to provision and manage your PostgreSQL instances offers several key benefits. Your database configurations become documented as
code, making it easy to replicate instances across development, staging, and production environments. You can also version control your database infrastructure alongside your application code and resource keys and user credentials can be managed securely and consistently. In addition, you can automate the entire database
lifecycle including provisioning, scaling, backup configurations, and user management. This approach is particularly valuable when managing multiple PostgreSQL instances or when you need to maintain consistent configurations across your organization.

Before you begin, ensure you have the following:

- An [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com/registration){: external} with appropriate permissions to create database instances.
- Access to a resource group in your {{site.data.keyword.cloud}} account.
- Terraform installed on your local machine (version 1.0 or later recommended).

### Install Terraform
{: #tf_install}
{: terraform}

If you don't already have Terraform installed, download and install it from

* [Step 1: Before you begin](#tf_prereqs)
* [Step 2: Configure {{site.data.keyword.cloud_notm}} provider](#tf_configure_provider)
* [Step 3: Project structure](#tf_project_structure)
* [Step 4: Configuration files](#tf_configuration_files)
* [Step 5: Deployment](#tf_deployment)
* [Step 6: Post-deployment](#tf_post_deployment)
* [Step 7: Connect to your database](#tf_connect_database)
