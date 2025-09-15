---

copyright:
  years: 2019, 2025
lastupdated: "2025-09-15"

keywords: postgresql, databases, read-only replica, resync, promote, cross-region replication, postgres replica, postgresql replica, leader deployment, read replica, data member, replication status

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# The read-only replica
{: #read-only-replicas}

A {{site.data.keyword.databases-for-postgresql}} read-only replica replicates all of your data from the leader deployment to the replica deployment through asynchronous replication. As the name implies, read-only replicas support read transactions and can be used to balance databases that have both write-heavy and read-heavy operations. The read-only replica has a single PostgreSQL data member, and it is billed at the [same per member consumption rates as the leader](https://{DomainName}/catalog/services/databases-for-postgresql/).

## The high-availability read-only replica
{: #the-ha-read-only-replica}

A high-availability {{site.data.keyword.databases-for-postgresql}} read-only replica provides benefits such as improved read scalability, increased availability, reduced read latency, backup and disaster recovery capabilities, and the ability to distribute read traffic efficiently. It contributes to a more robust and responsive database infrastructure for your application. For more information, see [The {{site.data.keyword.databases-for-postgresql}} high-availability read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-the-ha-read-only-replica&interface=ui).

## The leader
{: #read-only-replicas-leader}

On the _Read-only replicas_ tab of a {{site.data.keyword.databases-for-postgresql}} deployment before any read-only replicas are provisioned, the center pane notes that no read replicas exist and provides a **Create** button.

If a deployment is a leader and has a read-only replica that is already attached to it, then the _Replication_ pane has a list of replica deployments and a link to each one. Click the cog icon to the right of the read-only replica's deployment name to manage it.

## Provisioning a read-only replica
{: #read-only-replicas-provision}

Resources for PostgreSQL deployments are allocated per-deployment, and normal deployments have two members. Since a read-only replica has only one member, and provisioning currently uses values that are half of the requested values for memory and storage, provisioning can fail. The web UI cannot modify the value for storage and it automatically uses the leader deployment’s value - which is cut in half. If that is insufficient for your data to fit, you need to use the API or CLI to specify twice the storage you want to be provisioned. (The same applies to memory, although a lower amount of memory might not prevent the restore from being successful.) An update is in progress to remediate this situation.
{: important}

### Provisioning through the UI
{: #read-only-replicas-leader-ui}
{: ui}

Provision a read-only replica from the leader's _Read replicas_ tab by clicking **Create read-only replica**. The source instance is automatically completed. The read-only replica's name is auto-generated in the _Service name_ field, but you can rename it freely. You can choose the region to deploy it in, and its initial memory allocation. Disk size, version, and public or private endpoints are automatically configured to match the settings of the leader deployment.

If you use [Key Protect](/docs/cloud-databases?topic=cloud-databases-key-protect&interface=ui), Bring Your Own Key (BYOK) is supported only when provisioning from the CLI and API. Otherwise, the read-only replica is encrypted with a generated key.
{: .tip}

### Provisioning through the CLI
{: #read-only-replicas-leader-cli}
{: cli}

Provisioning a read-only replica through the CLI and the API works similarly to [provisioning a standard {{site.data.keyword.databases-for-postgresql}} deployment](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=ui). Provisioning is handled by the Resource Controller, and it uses a parameter `{"remote_leader_id": "crn:v1:..."}` to specify the leader of the replica you are provisioning.


To provision a read-only replica through the CLI, use a command like:

```sh
ibmcloud resource service-instance-create <REPLICA_NAME_OR_CRN> databases-for-postgresql standard <REGION> \
-p \ '{
  "remote_leader_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71819::",
  "members_memory_allocation_mb": "2048",
  "members_disk_allocation_mb": "10240"
}'
```
{: pre}

You must specify both the RAM and disk amounts, keeping in mind the minimum size is 2 GB RAM and 10 GB disk. You can optionally specify whether the read-only replica uses public or private endpoints. You are not able to specify a version for the read-only replica. The version is automatically set to the same major version as the leader deployment.

### Provisioning through the API
{: #read-only-replicas-leader-api}
{: api}

Provisioning a read-only replica through the API works similarly to [provisioning a standard {{site.data.keyword.databases-for-postgresql}} deployment](/docs/databases-for-postgresql?topic=databases-for-postgresql-provisioning&interface=api). Provisioning is handled by the Resource Controller, and it uses a parameter `{"remote_leader_id": "crn:v1:..."}` to specify the leader of the replica you are provisioning.

To provision a read-only replica through the API, use a command like:

```sh
curl -X POST \
  https://resource-controller.cloud.ibm.com/v2/resource_instances \
  -H 'Authorization: Bearer <>' \
  -H 'Content-Type: application/json' \
    -d '{
    "name": "<REPLICA_NAME_OR_CRN>",
    "target": "<REGION>",
    "resource_group": "<RESOURCE_GROUP_ID>",
    "resource_plan_id": "databases-for-postgresql-standard",
    "parameters": {
      "remote_leader_id": "crn:v1:bluemix:public:databases-for-postgresql:us-south:a/54e8ffe85dcedf470db5b5ee6ac4a8d8:1b8f53db-fc2d-4e24-8470-f82b15c71819::",
      "members_memory_allocation_mb": "2048",
      "members_disk_allocation_mb": "10240"
    }
  }'
```
{: pre}

You must specify both the RAM and disk amounts, keeping in mind the minimum size is 2 GB RAM and 10 GB disk. You can optionally specify whether the read-only replica uses public or private endpoints. You are not able to specify a version for the read-only replica. The version is automatically set to the same major version as the leader deployment.

## The read-only replica
{: #the-read-only-replica}

On the _Read replicas_ tab of a read-only replica, _Replication_ contains its name and region, and the name and region of its leader. It also has buttons to resync the read-only replica and to promote it.

### Checking replication status
{: #checking-replication-status}

You must monitor replication as replication status is not automatically monitored.

Check the replication status of a read-only replica with `psql`, but only from its leader. [Connect to the leader deployment with `psql`](/docs/databases-for-postgresql?topic=databases-for-postgresql-connecting-psql) using the [admin credentials](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management#the-admin-user). Once you are connected run either,
- For PostgreSQL version 10 and newer, `SELECT * from pg_stat_replication;`.

Or
- For PostgreSQL version 9.x and older, `SELECT * FROM get_pg_stat_replication();`.

### Read-only replica users and privileges
{: #read-only-replica-users-priv}

- Any user on the leader, even ones present before read-only replica provision, can log in to and run reads on a read-only replica with the same privileges to objects that they have on the leader.

- If you have more than one read-only replica that is attached to a leader, a user that is created on the leader is also created on all of the other read-only replicas.

- Users that are created on the leader persist on the read-only replica when it is promoted to a stand-alone deployment, including the `admin` user. When the read-only replica is promoted the users and privileges for all users on the leader are transferred to the promoted deployment.

- Write operations on the read-only replica for all users are not filtered or rejected, but fail at the database level.

You can also create users with access to the read-only replica and no access to the leader from the read-only replica. If you have more than one read-only replica that is attached to a leader, a user that is created on any one of the read-only replicas is also created on all of the other read-only replicas.

Read-only replica users who are created on a read-only replica are able connect to the replicas and run reads. Read-only replica users are not able to connect and run operations on the leader. They also do not persist when a read-only replica is promoted to a stand-alone deployment.

Read-only replica created users are assigned privileges by the leader, and are assigned the `ibm-cloud-base-user-ro` role, and are members of the `ibm-cloud-base-user` group. They have access to all of the objects that are created by other members of this group, including any users on the leader that were created through _Service Credentials_, the CLI, or the API. Consistent with privileges of the `ibm-cloud-base-user`, a read-only replica-created user does not have access to objects created by the admin user, or other users created through `psql`. For more information, see the [PostgreSQL Roles and Privileges](/docs/databases-for-postgresql?topic=databases-for-postgresql-user-management) page.

## Resyncing a read-only replica
{: #resyncing-read-only-replica}

If you need to resync a read-only replica, click the **Resync read-only replica** button. Resyncing is a disruptive operation and performing a resync tears down and rebuilds the data in the read-only replica. The read-only replica is not able to perform any other operations or run any queries while a resync is running. Queries are not rerouted to the leader, so any connections to the read-only replica fail until it is finished resyncing.

The amount of time it takes to resync a read-only replica varies, but the process can be long running.
{: .tip}

### Resyncing a Read-only Replica through the CLI
{: #resyncing-read-only-replica-cli}
{: cli}

To start a resync through the CLI, use the [`cdb read-replica-resync`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#read-replica-resync) command.
```sh
ibmcloud cdb read-replica-resync <DEPLOYMENT_NAME_OR_CRN>
```
{: pre}

### Resyncing a Read-only Replica through the API
{: #resyncing-read-only-replica-api}
{: api}

To start a resync through the API, send a POST to the [`/deployments/{id}/remotes/resync`](https://cloud.ibm.com/apidocs/cloud-databases-api#resync-read-only-replica) endpoint.
```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/remotes/resync \
  -H 'Authorization: Bearer <>'
```
{: pre}

## Promoting a read-only Replica
{: #promoting-read-only-replica}

A read-only replica is able to be promoted to an independent cluster that can accept write operations as well as read operations. If something happens to the leader deployment, the read-only replica can be promoted to a stand-alone cluster and start accepting writes from your application. Promoting a read-replica cluster to a stand-alone cluster will be quicker if the read-replica cluster already has more than one data member.

Upon promotion, the read-only replica terminates its connection to the leader and becomes a stand-alone {{site.data.keyword.databases-for-postgresql}} deployment. The deployment can start accepting and running read and write operations, backups are enabled, and it is issued its own admin user. A new data member is added so the deployment becomes a cluster with two data members. This increases the cost as it is billed at the same per member consumption rate, but the deployment has two members instead of one.

When you promote a read-only replica, you can skip the initial backup that would normally be taken upon promotion. Skipping the initial backup means that your replica becomes available more quickly, but there is no immediate backup available. You can start an on-demand backup once the promotion process is complete.

Once a read-only replica is promoted to an independent deployment, it is not possible to revert it back to a read-only replica or have it rejoin a leader.

## Promoting a read-only replica in the UI
{: #promoting-read-only-replica-ui}
{: ui}

To promote a read-only replica from the UI, click **Promote read-only replica**.

## Promoting a read-only replica in the CLI
{: #promoting-read-only-replica-cli}
{: cli}

To promote through the CLI, use the [`cdb read-replica-promote`](/docs/databases-cli-plugin?topic=databases-cli-plugin-cdb-reference#read-replica-promote) command.
```sh
ibmcloud cdb read-replica-promote <DEPLOYMENT_NAME_OR_CRN>
```
{: pre}

## Promoting a read-only replica through the API
{: #promoting-read-only-replica-api}
{: api}

To promote through the API, send a POST to the [`/deployments/{id}/remotes/promotion`](https://cloud.ibm.com/apidocs/cloud-databases-api#modify-read-only-replication-on-a-deployment) endpoint.
```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{"promotion": {}}' \
```
{: pre}

To promote and skip the initial backup after the promotion, also set `skip_initial_backup` in the JSON body.
```sh
curl -X POST \
  https://api.{region}.databases.cloud.ibm.com/v4/ibm/deployments/{id}/remotes/promotion \
  -H 'Authorization: Bearer <>'  \
 -H 'Content-Type: application/json' \
 -d '{"promotion": {"skip_initial_backup": true}}' \
 ```
 {: pre}

### Time to completion
{: #read-only-replica-completion}

The promote task completes only when the database is highly available. However, read/write availability occurs after about 10 minutes with one major caveat: the database is not highly available until the task completes.

The full promotion time of a read-replica is determined by the size of the data in two possible ways:
- Read replicas are single members. When promoted, the formation spec is changed to two members, which creates a second replica. The creation time of that replica depends on the size of the data. The creation of that replica runs at 25 MB/s to avoid saturating the network. As databases grow, the creation can take a substantial amount of time. The task does not complete until the creation of that replica is done.
- If you choose to take a backup as part of the promotion, the completion of that backup also needs to finish before the task completes. Again, this depends on the size of the database.

There is no High-Availability member until the promotion task completes. Likewise, if you have selected to have an initial backup, no backup exists until the second point completes or a manual backup is created.
{: note}

### Upgrading while promoting
{: #read-only-replica-upgrading-promoting}

If you need to upgrade to a new major version of the database, you can do so when promoting a read-only replica to a stand-alone deployment. For more information, see [Upgrading to a new major version](/docs/databases-for-postgresql?topic=databases-for-postgresql-upgrading).

## Read-only replica considerations
{: #read-only-replicas-consider}

- The read-only replica can exist in the same region as the source formation or in different one, enabling your data to be replicated across regions.

- A read-only replica must be the same major version as its leader.

- Backups are disabled on read-only replicas. Backups are taken only on leader deployments.

- Replicas are restorable to other regions, except for EU Cloud-enabled regions (currently `eu-de` and `par-01`), which are only restorable with each other (for example, `par-01` replicas can be restored to `eu-de`, and vice versa).

- There is a limit of five read-only replicas per leader.

- The read-only replica does not participate in leader->follower elections for the leader cluster and failover to the read-only replica is not automated. Promotion of the read-only replica to a full deployment is a manual, user-initiated task.

- The minimum size of a read-only replica is 2 GB RAM and 10 GB of disk. This is true even if your leader deployment is smaller.

- Read-only replicas do not auto-scale to match the leader. If the amount of data you store outgrows the disk that is allocated to your deployments, scale the disk on the read-only replicas and then the leader. Scaling the read-only replica first ensures that you do not run out of space on the read-only replicas. If you scaled the leader's disk for performance and not for space, it is not necessary to scale the read-only replicas.

- Replication is asynchronous, and might be subject to replication lag. By default, there is no consistent communication between the primary and replica. It is possible for a read-only replica to fall far enough behind that it needs to be resynced. Replication lag can be greater when the replica is in a region far away geographically from its leader.

- A read-only replica is a deployment with single data member and does not have any internal high availability. It is prone to temporary interruptions and downtime during maintenance. If you have applications that rely on read-only replicas, be sure to have logic to retry failed queries, or load-balancing over multiple read-only replicas.
