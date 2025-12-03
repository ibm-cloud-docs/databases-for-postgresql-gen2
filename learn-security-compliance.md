---

copyright:
  years: 2025
lastupdated: "2025-12-03"

keywords: postgresql, databases, soc, hipaa, gdpr, terms, postgresql security, Gen 2

subcollection: databases-for-postgresql-gen2

---

{{site.data.keyword.attribute-definition-list}}

# Security and compliance
{: #security-compliance}

[Gen 2]{: tag-purple}

{{site.data.keyword.databases-for}} Gen 2 is currently in Beta. The Beta plan is provided exclusively for evaluation and testing purposes. It is not covered by warranties, SLAs, or support, and is not intended for production use. For more information, see the [Beta reference](/docs/cloud-databases-gen2?topic=cloud-databases-gen2-icd-gen2-beta).
{: beta}

## Protection against unauthorized access
{: #protect-unauth}

{{site.data.keyword.databases-for-postgresql_full}} use the following methods to protect data in transit or in storage.
- All {{site.data.keyword.databases-for-postgresql}} connections use TLS/SSL encryption for data in transit. The current supported version of this encryption is TLS 1.2.
- Access to the Account, Management Console UI, and API is secured via [Identity and Access Management (IAM)](/docs/databases-for-postgresql?topic=databases-for-postgresql-iam).
- Access to the database is secured through the standard access controls provided by the database. These access controls are configured to require valid database-level credentials that are obtainable only through prior access to the database or through our Management Console UI or API.
- All {{site.data.keyword.databases-for-postgresql}} storage is provided on storage encrypted with LUKS using AES-256. The default keys are managed by [{{site.data.keyword.keymanagementserviceshort}}](/docs/key-protect?topic=key-protect-about). Bring-your-own-key (BYOK) for encryption is also available through [Key Protect Integration](/docs/cloud-databases?topic=cloud-databases-key-protect&interface=ui).
- CBR (Context-based restriction) - Protect your {{site.data.keyword.databases-for}} resource with [Context-based restriction (CBR)](/docs/cloud-databases?topic=cloud-databases-cbr&interface=ui).
- Private networking - {{site.data.keyword.databases-for-postgresql}} is integrated with [Service endpoints](/docs/databases-for-postgresql?topic=databases-for-postgresql-service-endpoints&interface=ui). 


## Data resilience
{: #data-resilience}

- [Backups](/docs/cloud-databases?topic=cloud-databases-dashboard-backups&interface=ui) are included in the service. {{site.data.keyword.databases-for-postgresql}} backups reside in [{{site.data.keyword.cos_full_notm}}](/docs/cloud-object-storage?topic=cloud-object-storage-about-cloud-object-storage&cloud-object-storage-about-cloud-object-storage) and are also [encrypted](/docs/cloud-object-storage?topic=cloud-object-storage-security).
- {{site.data.keyword.databases-for-postgresql}} deployments are configured with replication. Deployments contain a cluster with two data members. Both members contain a copy of your data by using asynchronous replication, with a distributed consensus mechanism to maintain cluster state and handle failovers. 
- If you deploy to an {{site.data.keyword.cloud_notm}} Single-Zone Region (SZR), each database member resides on a different host in the data center. 
- If you deploy to an {{site.data.keyword.cloud_notm}} Multi-Zone Region (MZR), the members are spread over the region's availability zone locations. 

## SOC 2 Type 2 certification
{: #soc-type2-cert}

{{site.data.keyword.IBM_notm}} provides a Service Organization Controls (SOC) 2 Type 2 report for {{site.data.keyword.databases-for-postgresql}}. The reports evaluate IBM's operational controls according to the criteria set by the American Institute of Certified Public Accountants (AICPA) Trust Services Principles. The Trust Services Principles define adequate control systems and establish industry standards for service providers such as IBM Cloud to safeguard their customers' data and information.

You can request an SOC 2 Type 2 report from the customer portal or contact your sales representative. Alternatively, you can open a support ticket with [IBM Cloud support](https://cloud.ibm.com/unifiedsupport/supportcenter){: .external}

## ISO 27017, ISO 27018
{: #iso-27017-27018}

{{site.data.keyword.databases-for-postgresql}} conforms to the guidelines for information security controls applicable to the provision and use of cloud services as defined in [ISO 27017](https://www.iso.org/standard/43757.html){: .external} and [ISO 27018](https://www.iso.org/standard/76559.html){: .external}.

## General Data Protection Regulation (GDPR) 
{: #gdpr}

If you have an account with IBM Cloud, your personal data is held by {{site.data.keyword.cloud_notm}}. The {{site.data.keyword.IBM_notm}} Data Processing Addendum (DPA) applies to the processing of client's personal data by {{site.data.keyword.IBM_notm}} on behalf of client to provide {{site.data.keyword.IBM_notm}} standard services.  
[IBM DPA](https://www.ibm.com/support/customer/csol/terms/?id=Z126-7870&lc=en){: .external}

{{site.data.keyword.databases-for-postgresql}} processes limited client Personal Information (PI) in the course of running the service and optimizing the user experience. 

{{site.data.keyword.databases-for-postgresql}} provides a [Data Sheet Addendum (DSA)](https://www.ibm.com/software/reports/compatibility/clarity-reports/report/html/softwareReqsForProduct?deliverableId=CD09D2E06DC811E8A0B560E89C071ECC){: .external} with its policies as a Data Processor regarding content and data protection. 

## HIPAA
{: #hipaa}

{{site.data.keyword.databases-for-postgresql}} meets the required {{site.data.keyword.IBM_notm}} controls that are commensurate with the Health Insurance Portability and Accountability Act of 1996 (HIPAA) Security and Privacy Rule requirements. These requirements include the appropriate administrative, physical, and technical safeguards required of Business Associates in 45 CFR Part 160 and Subparts A and C of Part 164. HIPAA must be requested at the time of provisioning and requires a representative to sign a Business Associate Addendum (BAA) agreement with {{site.data.keyword.IBM_notm}}.

## PCI DSS
{: #pci-dss}

{{site.data.keyword.databases-for-postgresql}} are compliant with the Payment Card Industry Data Security Standard (PCI DSS). {{site.data.keyword.cloud_notm}} completes annual PCI DSS assessments by using an approved Qualified Security Assessor (QSA), and the resulting Attestations of Compliance (AOCs) and Service Responsibility Matrix (SRM) guides are available upon customer request. Auditors reviewed {{site.data.keyword.databases-for-postgresql}} for compliance under PCI DSS version 3.2.1 at Service Provider Level 1. 

Customers are responsible for the storing, processing, and transmission of their cardholder data, and can create cardholder data environments (CDEs) that can store, transmit, or process cardholder data by using {{site.data.keyword.databases-for-postgresql}}. Customers can request and use the {{site.data.keyword.cloud_notm}} AOCs and SRM guides when they seek their own PCI DSS certifications. It is the responsibility of the customer to document and operate CDEs and applications that are built by using {{site.data.keyword.cloud_notm}} Platform services in a PCI DSS-compliant manner.

It is the customer’s responsibility to familiarize themselves with these processes and to manage data retention and removal from the service according to the customer’s policies.

A full list of PCI DSS-ready {{site.data.keyword.cloud_notm}} Platform services, and options to request a PCI DSS AOC and SRM guide, can be found at the [IBM Cloud compliance page](https://www.ibm.com/cloud/compliance/industry).

## Terms
{: #terms}

- [The IBM Privacy Policy](https://www.ibm.com/privacy/us/en/)
- [The IBM Cloud Notices and Terms of Use](/docs/overview/terms-of-use?topic=overview-terms)
