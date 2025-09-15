---

copyright:
   years: 2023
lastupdated: "2023-11-17"

keywords: postgresql, pgadmin

subcollection: databases-for-postgresql

content-type: tutorial
account-plan: paid
completion-time: 30min

---

{{site.data.keyword.attribute-definition-list}}

# Deploy pgadmin using {{site.data.keyword.codeengineshort}} and connect to your {{site.data.keyword.databases-for-postgresql}} instance
{: #pgadmin-code-engine-icd-postgresql}
{: toc-content-type="tutorial"}
{: toc-completion-time="30min"}

With this tutorial, deploy [pgadmin](https://www.pgadmin.org/){: external} using [{{site.data.keyword.codeengineshort}}](https://www.ibm.com/products/code-engine){: external} and connect to your [{{site.data.keyword.databases-for-postgresql}}](https://www.ibm.com/products/databases-for-postgresql){: external} instance. pgadmin is a web interface that allows you to view and modify the data in your PostgreSQL database. {{site.data.keyword.codeengineshort}} is a a fully managed, serverless platform that allows you to run workloads without worrying about deploying infrastructure. PostgreSQL is an open source database that has a strong reputation for its reliability, flexibility, and support of open technical standards.
{: shortdesc}

{{site.data.keyword.codeengineshort}} is a paid-for service, so following this tutorial will incur charges.
{: note}

## Before you start
{: #pgadmin-code-engine-icd-postgresql-before-start}

Before you begin, ensure you have the following:

- An [{{site.data.keyword.cloud_notm}} Account](https://cloud.ibm.com/registration){: external}.
- [Terraform](https://www.terraform.io/){: external} - to deploy infrastructure
- A [{{site.data.keyword.databases-for-postgresql}} instance](https://cloud.ibm.com/databases/databases-for-postgresql/create){: external}

## Obtain an API key to deploy infrastructure to your account
{: #pgadmin-code-engine-icd-postgresql-obtain-key}
{: step}

Follow [these steps](/docs/account?topic=account-userapikey&interface=ui#create_user_key){: external} to create an {{site.data.keyword.cloud_notm}} API key that enables Terraform to provision infrastructure into your account. You can create up to 20 API keys.

For security reasons, the API key is only available to be copied or downloaded at the time of creation. If the API key is lost, you must create a new API key.
{: note}

## Clone the project
{: #pgadmin-code-engine-icd-postgresql-clone-project}
{: step}

```sh
git clone https://github.com/IBM/ibm-postgresql-pgadmin-codeengine.git
```
{: pre}

## Install the infrastructure
{: #pgadmin-code-engine-icd-postgresql-install-infra}
{: step}

1. Navigate into the `terraform` folder of the cloned project.

   ```sh
   cd ibm-postgresql-pgadmin-codeengine/terraform
   ```
   {: pre}

1. On your machine, create a document that is named `terraform.tfvars`, with the following fields:

   ```sh
    ibmcloud_api_key = "<your_api_key_from_step_1>"
    region = "<your_region>"
    pg_admin_username = "<username_for_pgadmin>" (has to be an email address e.g. user@domain.com)
      pg_admin_password = "<a_password_for_the_pgadmin_user>"
      pg_user = "<database_user>"
      pg_password = "<database_user_password"
      pg_host = "<host_of_your_postgres_instance>" (e.g. something.databases.appdomain.cloud)
      pg_port = "<instance_port>"
   ```
   {: pre}

   The `terraform.tfvars` document contains variables that you might want to keep secret.
   {: important}

1. Install the infrastructure with the following command:

   ```sh
   terraform init
   terraform apply --auto-approve
   ```
   {: pre}

## Visit your pgadmin deployment
{: #pgadmin-code-engine-icd-postgresql-visit-deployment}
{: step}

The previous step produces a URL, which is the public URL of your pgadmin deployment. It looks something like: `https://pgadmin-app.1834dcfgrtygbg.eu-gb.codeengine.appdomain.cloud`.

Visit that URL in your web browser. You should see the pgadmin login screen where you can log in with the pgadmin credentials you defined above.
Once you are logged in, connect to the PostgreSQL database using the [Connect to Server](https://www.pgadmin.org/docs/pgadmin4/development/connecting.html){: external} dialog. You need the `pg_host`, `pg_port`, `pg_user` and `pg_password` defined above. You can set the `ssl mode` value in the `Parameters` section to `Allow` to override certificate validation (this is acceptable for testing purposes, but you may need a more secure connection for production deployments). You can now access your postgresql deployment!

This tutorial incurs some Code Engine charges. After you finish this tutorial, remove all the infrastructure by going to the `terraform` directory of the project and using the command:

```sh
terraform destroy
```
{: pre}
