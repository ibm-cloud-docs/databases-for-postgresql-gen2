---

copyright:
  years: 2020, 2023
lastupdated: "2023-06-07"

keywords: databases, scaling, horizontal scaling, postgresql members, postgres members, postgres scaling, data members

subcollection: databases-for-postgresql

---

{{site.data.keyword.attribute-definition-list}}


# Adding PostgreSQL members
{: #horizontal-scaling}

By adding more members, it's possible to scale your {{site.data.keyword.databases-for-postgresql_full}} deployment horizontally, which increases deployment reliability by spreading data across extra `Availability zones`, where available. Horizontally scaling deployments to three PostgreSQL members is required before enabling [synchronous replication](/docs/databases-for-postgresql?topic=databases-for-postgresql-changing-configuration#general-settings). Adding a member does not spread loads nor help with capacity in your deployment; however, PostgreSQL members are eligible for failovers.

Horizontal scaling can increase only disk and memory allotments. Members cannot be scaled down.
{: .note}

Members that you add to your deployment are added with the amount of disk, memory, and CPU as the other members currently in your deployment. A visual representation of your data members and their resource allocation is available on the _Resources_ tab of your deployment's _Manage_ page. However, horizontal scaling is only available by using the API.
{: note}

{{site.data.keyword.databases-for-postgresql}} deployments have two data members in a cluster, and resources are allocated to both members equally. For example, the minimum storage of a PostgreSQL deployment is 10240 MB, which equates to an initial size of 5120 MB per member. The minimum RAM for a PostgreSQL deployment is 8192 MB, which equates to an initial allocation of 4096 MB per member.

Billing is based on the _total_ amount of resources that are allocated to the service. 
{: .tip}

## Adding members through the API
{: #adding-members-api}

The _Foundation Endpoint_ that is shown in the _Overview_ of your service provides the base URL to access this deployment through the API.

To view the current and scalable resources on a deployment, use the [/deployments/{id}/groups](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#listdeploymentscalinggroups-permissions) endpoint.

```sh
curl -X GET https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups -H 'Authorization: Bearer <>' \
```
{: pre}

To add members, use the [/deployments/{id}/groups/{group_id}](https://cloud.ibm.com/apidocs/cloud-databases-api/cloud-databases-api-v5#setdeploymentscalinggroup-permissions) endpoint. This endpoint sends a `PATCH` request with the number of members that you want in your deployment. The following example request increases the number of members from the default of 2 to 3.

A 2-member deployment can be scaled only up to a 3-member instance.
{: note}

```sh
curl -X PATCH https://api.{region}.databases.cloud.ibm.com/v5/ibm/deployments/{id}/groups/{group_id} 
-H 'Authorization: Bearer <>' 
-H 'Content-Type: application/json' 
-d '{"members": {"allocation_count": 3}}' \
```
{: pre}
