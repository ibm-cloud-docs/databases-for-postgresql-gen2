---

copyright:
  years: 2023
lastupdated: "2023-10-10"

keywords: postgresql, databases, ha read-only replica, high availability read-only replica, resync, promote, cross-region replication, postgres replica, postgresql replica, leader deployment, read replica, data member, replication status, url encoding, foundation endpoint

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}

# The high-availability read-only replica
{: #the-ha-read-only-replica}

A high-availability {{site.data.keyword.databases-for-postgresql}} read-only replica provides benefits such as improved read scalability, increased availability, reduced read latency, backup and disaster recovery capabilities, and the ability to distribute read traffic efficiently. It contributes to a more robust and responsive database infrastructure for your application.

## Provision a high-availability read-only replica
{: #provision-ha-read-only-replica}

High-availability {{site.data.keyword.databases-for-postgresql}} read-only replicas must initially be provisioned in the single-member (non-HA) configuration. To provision a read-only replica, follow the procedure that is outlined in [Provisioning a read-only replica](/docs/databases-for-postgresql?topic=databases-for-postgresql-read-only-replicas&interface=ui#read-only-replicas-provision).

After your read-replica is provisioned, use the [{{site.data.keyword.databases-for}} API](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#introduction) to scale up the read replica to two (or more) members.

To scale up your read-replica, use the [Scaling groups endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#setdeploymentscalinggroup) of the {{site.data.keyword.databases-for}} API.

### The high-availability read-only replica command
{: #provision-ha-read-only-replica-command}

To execute the command, you first need the `{foundation endpoint}` and the `{id}` for your {{site.data.keyword.databases-for}} instance. 

The `{foundation endpoint}` is the starting point for accessing and interacting with the resources that are exposed by the {{site.data.keyword.databases-for}} API. The foundation endpoint is visible in the UI within your **Overview**. To use the foundation endpoint, it must first be URL encoded. URL encoding replaces unsafe characters with a `%` followed by two hexadecimal digits representing the character's ASCII code.

The `{id}` is your Cloud Resource Name (CRN) and uniquely identifies your {{site.data.keyword.databases-for}} instance. It is visible in the UI within your **Overview**.

To use your instance's `{CRN}` and `{foundation endpoint}` in a command, they must first be URL encoded. URL encoding replaces unsafe characters with a `%` followed by two hexadecimal digits representing the character's ASCII code.
{: note}

```sh
curl -XPATCH -H 'Authorization: Bearer <>' "{foundation_endpoint}/deployments/{id}/groups/member" -d '{"members": {"allocation_count": 2}}'
```
{: pre}


## Verifying the state of your read-replica
{: #verifying-ha-read-only-replica}

To verify the current state of your read-replica, use the [Scaling groups endpoint](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeploymentscalinggroups) of the {{site.data.keyword.databases-for}} API.

Use a command like:

```sh
curl -XGET -H 'Authorization: Bearer <>' "{foundation_endpoint}/deployments/{id}/groups"
```
{: pre}
