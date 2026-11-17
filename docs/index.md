# Fabric Mirroring: A Practical Guide

This guide covers the main mirroring patterns in Microsoft Fabric, source-specific setup, and open mirroring implementations. Use Part 1 for concepts and architecture, Part 2 for source guides and related application integrations, and Part 3 for open mirroring and partner-led integrations.

Each part has its own **chapter index** in a separate `readme.md`, listing only the chapters in that part. Every chapter links back to its part's index.

**FabCon September 2026:** the [announcement coverage record](history.md#8-october-2026-fabcon-announcement-coverage) links each relevant announcement to its chapter and distinguishes GA, Preview, Coming Soon, and unresolved documentation differences.

For eligible database and open mirrored replicas, Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity. This is not a blanket storage allowance for catalog shortcuts or every application integration; check the source guide's billing boundaries.

**Latest source additions:** [Chapter 30: Google Lakehouse Runtime Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/30%20-%20Google%20Lakehouse%20Runtime%20Catalog%20Mirroring.md) and [Chapter 31: Business Central Mirroring](Part%202-%20Source-Specific%20Mirroring%20Guides/31%20-%20Dynamics%20365%20Business%20Central%20Mirroring.md). SAP BDC remains a separate [Chapter 29](Part%202-%20Source-Specific%20Mirroring%20Guides/29%20-%20SAP%20Business%20Data%20Cloud%20Connect%20for%20Microsoft%20Fabric.md). The current edition has 48 consecutively numbered chapters.

[![Mirroring types and their outputs](assets/diagrams/index/diagram-01.png)](assets/diagrams/index/diagram-01.excalidraw.png)

* Database mirroring copies source data into Delta tables in OneLake.
* Metadata mirroring uses catalog or connection integrations and OneLake shortcuts to access source data in place.
* Open mirroring accepts developer-defined or partner-defined change data in a landing zone that Fabric processes.

## Table of Contents

### Part 1: Concepts and Architecture

**Part index:** [Chapters in Part 1](Part%201%20-%20Concepts%20and%20Architecture/readme.md)

* [Chapter 1: Introduction to Fabric Mirroring](Part%201%20-%20Concepts%20and%20Architecture/01%20-%20Introduction%20to%20Fabric%20Mirroring.md)
* [Chapter 2: Types of Mirroring in Fabric](Part%201%20-%20Concepts%20and%20Architecture/02%20-%20Types%20of%20Mirroring%20in%20Fabric.md)
* [Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts](Part%201%20-%20Concepts%20and%20Architecture/03%20-%20Methods%20of%20Mirroring%20-%20Push%2C%20Pull%20or%20Polling%2C%20and%20Shortcuts.md)
* [Chapter 4: The Anatomy of a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/04%20-%20The%20Anatomy%20of%20a%20Mirrored%20Database.md)
* [Chapter 5: Monitoring a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/05%20-%20Monitoring%20a%20Mirrored%20Database.md)
* [Chapter 6: Using the Fabric REST API](Part%201%20-%20Concepts%20and%20Architecture/06%20-%20Using%20the%20Fabric%20REST%20API.md)
* [Chapter 7: Deploying a Mirrored Database Using CI/CD](Part%201%20-%20Concepts%20and%20Architecture/07%20-%20Deploying%20a%20Mirrored%20Database%20Using%20CI-CD.md)
* [Chapter 8: Using a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/08%20-%20Using%20a%20Mirrored%20Database.md)
* [Chapter 9: Extended Capabilities](Part%201%20-%20Concepts%20and%20Architecture/09%20-%20Extended%20Capabilities.md)
* [Chapter 10: Billing and Capacity Management](Part%201%20-%20Concepts%20and%20Architecture/10%20-%20Billing%20and%20Capacity%20Management.md)

**Security reading path:** [Section 4.5](Part%201%20-%20Concepts%20and%20Architecture/04%20-%20The%20Anatomy%20of%20a%20Mirrored%20Database.md#45-security-of-a-mirrored-database) explains OneLake roles, DefaultReader, SQL identity modes, shortcuts, and access validation; [Chapter 8](Part%201%20-%20Concepts%20and%20Architecture/08%20-%20Using%20a%20Mirrored%20Database.md#choose-security-for-each-read-path) applies those choices to SQL, Spark, OneLake, and Power BI consumers.

### Part 2: Source-Specific Mirroring Guides

**Part index:** [Chapters in Part 2](Part%202-%20Source-Specific%20Mirroring%20Guides/readme.md)

**Start with the source, then its setup walkthrough.** Each guide takes you from source preparation and permissions through the Fabric connection, initial validation and operational troubleshooting. Do the source-administrator steps before opening the Fabric wizard; a successful sign-in does not prove that change capture or storage access is configured.

**Gateway review, 4 November 2026:** use the [source-by-source VNet and standard-mode on-premises gateway reference](appendix.md#gateway-support-by-source) before choosing a private-network design. It separates documented support from unpublished procedures and distinguishes gateway connection traffic from ongoing replication. The [release/operational baseline](appendix.md#gateway-releases-and-operational-baseline) covers supported versions, sign-in updates, subnet/identity requirements, diagnostics, and costs.

#### Setup and Troubleshooting Directory

Each setup link starts with the source-administrator work, not just the Fabric wizard. The common-problems sections distinguish documented behavior, public reports, and access-limited research leads.

| Source chapter | Practical setup | Common problems |
|---|---|---|
| [11: Azure SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/11%20-%20Azure%20SQL%20Database.md) | [Identity, SQL grants, networking, and first replication](Part%202-%20Source-Specific%20Mirroring%20Guides/11%20-%20Azure%20SQL%20Database.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/11%20-%20Azure%20SQL%20Database.md#public-issues-and-common-pitfalls) |
| [12: Azure SQL Managed Instance](Part%202-%20Source-Specific%20Mirroring%20Guides/12%20-%20Azure%20SQL%20Managed%20Instance.md) | [Choose the update-policy branch before configuring](Part%202-%20Source-Specific%20Mirroring%20Guides/12%20-%20Azure%20SQL%20Managed%20Instance.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/12%20-%20Azure%20SQL%20Managed%20Instance.md#public-issues-and-common-pitfalls) |
| [13: Azure Cosmos DB](Part%202-%20Source-Specific%20Mirroring%20Guides/13%20-%20Azure%20Cosmos%20DB.md) | [Continuous backup, credentials, and network path](Part%202-%20Source-Specific%20Mirroring%20Guides/13%20-%20Azure%20Cosmos%20DB.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/13%20-%20Azure%20Cosmos%20DB.md#public-issues-and-common-pitfalls) |
| [14: Azure Databricks / Unity Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/14%20-%20Azure%20Databricks%20%28Unity%20Catalog%29.md) | [External data access, UC grants, and separate ADLS access](Part%202-%20Source-Specific%20Mirroring%20Guides/14%20-%20Azure%20Databricks%20%28Unity%20Catalog%29.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/14%20-%20Azure%20Databricks%20%28Unity%20Catalog%29.md#public-issues-and-common-pitfalls) |
| [15: Azure Monitor](Part%202-%20Source-Specific%20Mirroring%20Guides/15%20-%20Azure%20Monitor.md) | [Workspace, source identity, and table permissions](Part%202-%20Source-Specific%20Mirroring%20Guides/15%20-%20Azure%20Monitor.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/15%20-%20Azure%20Monitor.md#public-issues-and-common-pitfalls) |
| [16: Google BigQuery](Part%202-%20Source-Specific%20Mirroring%20Guides/16%20-%20Google%20BigQuery.md) | [APIs, IAM, staging bucket, and source change history](Part%202-%20Source-Specific%20Mirroring%20Guides/16%20-%20Google%20BigQuery.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/16%20-%20Google%20BigQuery.md#public-issues-and-common-pitfalls) |
| [17: Oracle](Part%202-%20Source-Specific%20Mirroring%20Guides/17%20-%20Oracle.md) | [Archive logs, supplemental logging, grants, and gateway](Part%202-%20Source-Specific%20Mirroring%20Guides/17%20-%20Oracle.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/17%20-%20Oracle.md#public-issues-and-common-pitfalls) |
| [18: PostgreSQL](Part%202-%20Source-Specific%20Mirroring%20Guides/18%20-%20PostgreSQL.md) | [Server parameters, identity, ownership, and replica identity](Part%202-%20Source-Specific%20Mirroring%20Guides/18%20-%20PostgreSQL.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/18%20-%20PostgreSQL.md#public-issues-and-common-pitfalls) |
| [19: MySQL](Part%202-%20Source-Specific%20Mirroring%20Guides/19%20-%20MySQL.md) | [Binlogs, UAMI, connection grants, and table selection](Part%202-%20Source-Specific%20Mirroring%20Guides/19%20-%20MySQL.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/19%20-%20MySQL.md#public-issues-and-common-pitfalls) |
| [20: SAP](Part%202-%20Source-Specific%20Mirroring%20Guides/20%20-%20SAP.md) | [Extraction-route prerequisites, ADLS, and the Fabric mirror](Part%202-%20Source-Specific%20Mirroring%20Guides/20%20-%20SAP.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/20%20-%20SAP.md#public-issues-and-common-pitfalls) |
| [21: SharePoint List](Part%202-%20Source-Specific%20Mirroring%20Guides/21%20-%20SharePoint%20List.md) | [Site access, native connector, and supported lists](Part%202-%20Source-Specific%20Mirroring%20Guides/21%20-%20SharePoint%20List.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/21%20-%20SharePoint%20List.md#public-issues-and-common-pitfalls) |
| [22: Snowflake](Part%202-%20Source-Specific%20Mirroring%20Guides/22%20-%20Snowflake.md) | [Runtime role, change tracking, retention, and Iceberg branch](Part%202-%20Source-Specific%20Mirroring%20Guides/22%20-%20Snowflake.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/22%20-%20Snowflake.md#public-issues-and-common-pitfalls) |
| [23: SQL Server 2016-2022](Part%202-%20Source-Specific%20Mirroring%20Guides/23%20-%20SQL%20Server%202016%E2%80%932022.md) | [SQL Agent, CDC, setup/runtime permissions, and gateway](Part%202-%20Source-Specific%20Mirroring%20Guides/23%20-%20SQL%20Server%202016%E2%80%932022.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/23%20-%20SQL%20Server%202016%E2%80%932022.md#public-issues-and-common-pitfalls) |
| [24: SQL Server 2025](Part%202-%20Source-Specific%20Mirroring%20Guides/24%20-%20SQL%20Server%202025.md) | [Azure Arc, primary identity, SQL grants, and network paths](Part%202-%20Source-Specific%20Mirroring%20Guides/24%20-%20SQL%20Server%202025.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/24%20-%20SQL%20Server%202025.md#public-issues-and-common-pitfalls) |
| [25: Fabric SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/25%20-%20Fabric%20SQL%20Database.md) | [Automatic mirroring, permissions, and both query surfaces](Part%202-%20Source-Specific%20Mirroring%20Guides/25%20-%20Fabric%20SQL%20Database.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/25%20-%20Fabric%20SQL%20Database.md#public-issues-and-common-pitfalls) |
| [26: Dremio Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/26%20-%20Dremio%20Catalog%20Mirroring.md) | [Catalog authorization and storage credential vending](Part%202-%20Source-Specific%20Mirroring%20Guides/26%20-%20Dremio%20Catalog%20Mirroring.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/26%20-%20Dremio%20Catalog%20Mirroring.md#public-issues-and-common-pitfalls) |
| [27: AWS Glue Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/27%20-%20AWS%20Glue%20Catalog%20Mirroring.md) | [Glue/Lake Formation grants and S3 reads](Part%202-%20Source-Specific%20Mirroring%20Guides/27%20-%20AWS%20Glue%20Catalog%20Mirroring.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/27%20-%20AWS%20Glue%20Catalog%20Mirroring.md#public-issues-and-common-pitfalls) |
| [28: Dataverse Link to Microsoft Fabric](Part%202-%20Source-Specific%20Mirroring%20Guides/28%20-%20Dataverse%20Link%20to%20Microsoft%20Fabric%20and%20Low-Latency%20Sync.md) | [Source preparation, identity, tables, and linked Lakehouse](Part%202-%20Source-Specific%20Mirroring%20Guides/28%20-%20Dataverse%20Link%20to%20Microsoft%20Fabric%20and%20Low-Latency%20Sync.md#setup-walkthrough) | [Troubleshooting](Part%202-%20Source-Specific%20Mirroring%20Guides/28%20-%20Dataverse%20Link%20to%20Microsoft%20Fabric%20and%20Low-Latency%20Sync.md#troubleshooting) |
| [29: SAP Business Data Cloud Connect for Microsoft Fabric](Part%202-%20Source-Specific%20Mirroring%20Guides/29%20-%20SAP%20Business%20Data%20Cloud%20Connect%20for%20Microsoft%20Fabric.md) | [Availability and readiness gates](Part%202-%20Source-Specific%20Mirroring%20Guides/29%20-%20SAP%20Business%20Data%20Cloud%20Connect%20for%20Microsoft%20Fabric.md#readiness-checklist--not-an-executable-setup-procedure) | [Common misidentifications](Part%202-%20Source-Specific%20Mirroring%20Guides/29%20-%20SAP%20Business%20Data%20Cloud%20Connect%20for%20Microsoft%20Fabric.md#troubleshooting-and-common-misidentifications) |
| [30: Google Lakehouse Runtime Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/30%20-%20Google%20Lakehouse%20Runtime%20Catalog%20Mirroring.md) | [Federation, runtime catalog, and GCS permissions](Part%202-%20Source-Specific%20Mirroring%20Guides/30%20-%20Google%20Lakehouse%20Runtime%20Catalog%20Mirroring.md#setup-walkthrough) | [Issues and troubleshooting](Part%202-%20Source-Specific%20Mirroring%20Guides/30%20-%20Google%20Lakehouse%20Runtime%20Catalog%20Mirroring.md#public-issues-and-troubleshooting) |
| [31: Dynamics 365 Business Central Mirroring](Part%202-%20Source-Specific%20Mirroring%20Guides/31%20-%20Dynamics%20365%20Business%20Central%20Mirroring.md) | [Application registration, Open Mirroring, companies, and tables](Part%202-%20Source-Specific%20Mirroring%20Guides/31%20-%20Dynamics%20365%20Business%20Central%20Mirroring.md#setup-walkthrough) | [Documented pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/31%20-%20Dynamics%20365%20Business%20Central%20Mirroring.md#documented-pitfalls) |

> **Note:** Chapters 14, 15, 26, 27, and 30 cover metadata mirroring. Chapters 21 and 22 also describe shortcut-backed paths alongside replicated tables. Chapters 15, 19, 26, 27, and 30 cover Public Preview sources. Google Lakehouse Runtime has its own chapter and is separate from both AWS Glue and BigQuery.

Chapters 28 and 29 cover related source-managed integrations: Dataverse maintains the Delta replica and manages its Lakehouse shortcuts, while SAP BDC Connect is an announced data-product sharing path distinct from Chapter 20's Datasphere/ADLS replication. They are full source chapters, without assuming identical native Mirroring item types or operating rules. The SAP BDC chapter explicitly records the absence of a verified Fabric-specific GA/setup confirmation.

Chapter 31 follows Business Central's own assisted setup for an application-managed Open Mirroring publisher. It includes the source prerequisites, Entra registration, company/table selection, synchronization controls, monitoring, and recovery cautions. Its documentation still carries prerelease and conflicting preview-rollout wording; Open Mirroring being GA does not make the Business Central integration GA. Dataverse low-latency sync is GA-announced with station-based rollout, while its optional SQL endpoint metadata sync remains Preview.

#### Before You Start a Setup Walkthrough

| Prepare | Why it matters |
|---|---|
| Source product, version, edition and deployment type | Azure SQL Database is not SQL Server; a catalog shortcut is not database replication. Use the matching source-specific path. |
| Source administrator, Fabric implementer and identity/network owners | Source configuration, the connection credential, the publishing identity and the analyst's access can be different responsibilities. |
| Approved change window and recovery plan | Enabling capture, changing logging/backup settings, restarting a source or reseeding tables can affect production. Review every command and substitute your own identifiers. |
| A small representative test object | Prove initial data and supported changes before scaling up. Make mutations only in approved test data, not in production application tables. For catalog mirroring, prove both metadata visibility and underlying storage reads. |
| Ongoing owners and monitoring | Record credential expiry, source-log/storage retention, connection ownership, capacity state and source versus Fabric monitoring responsibilities. |

**How to read the evidence:** Microsoft Learn defines documented requirements and limits. Dated Reddit and other community reports are troubleshooting leads, not proof that an old defect still exists or that a workaround is supported. Search-index-only leads are labelled separately when a site blocks direct access; an indexed title or snippet does not verify a thread's replies or resolution. Where the public setup documentation is incomplete or inconsistent, the chapter says so rather than inventing steps.

**Documentation images:** setup screenshots are local copies of images from Microsoft's public documentation. Each caption links to the original image and article, credits Microsoft, states the licence and identifies whether the image was changed. Where a source tutorial has no screenshots, a relevant shared monitoring or shortcut screenshot is explicitly labelled as such, not presented as that source's wizard. Historical UI labels and preview badges may differ from your tenant; the linked current documentation takes precedence.

### Part 3: Open Mirroring

**Part index:** [Chapters in Part 3](Part%203%20-%20Open%20Mirroring/readme.md)

**Start with the source, then choose what to own.** Chapters 32-37 explain the shared contract and engineering responsibilities. Chapters 38-47 examine actual GitHub implementations and tools; Chapter 48 compares them and keeps blog-only references separate. Open-source code is not a promise of production support, and does not make Fabric capacity, the source system, or publisher hosting free.

#### Foundations and shared implementation guidance

* [Chapter 32: What is Open Mirroring and Why It's Useful](Part%203%20-%20Open%20Mirroring/32%20-%20What%20is%20Open%20Mirroring%20and%20Why%20It%27s%20Useful.md)
* [Chapter 33: Setting Up Open Mirroring: Step-by-Step Configuration](Part%203%20-%20Open%20Mirroring/33%20-%20Setting%20Up%20Open%20Mirroring%20-%20Step-by-Step%20Configuration.md)
* [Chapter 34: Code Samples and the Fabric Toolbox](Part%203%20-%20Open%20Mirroring/34%20-%20Code%20Samples%20and%20the%20Fabric%20Toolbox.md)
* [Chapter 35: Use Cases and Examples](Part%203%20-%20Open%20Mirroring/35%20-%20Use%20Cases%20and%20Examples.md)
* [Chapter 36: Metadata and Change Files](Part%203%20-%20Open%20Mirroring/36%20-%20Metadata%20and%20Change%20Files.md)
* [Chapter 37: Common Issues and Troubleshooting](Part%203%20-%20Open%20Mirroring/37%20-%20Common%20Issues%20and%20Troubleshooting.md)

#### SDK and source-backed solution chapters

* [Chapter 38: The Microsoft Open Mirroring Python SDK](Part%203%20-%20Open%20Mirroring/38%20-%20The%20Microsoft%20Open%20Mirroring%20Python%20SDK.md)
* [Chapter 39: GenericMirroring - A Multi-Source C# Publisher](Part%203%20-%20Open%20Mirroring/39%20-%20GenericMirroring%20-%20A%20Multi-Source%20C%23%20Publisher.md)
* [Chapter 40: Toolbox Notebook Solutions - Excel, SharePoint, MySQL, and Snowflake](Part%203%20-%20Open%20Mirroring/40%20-%20Toolbox%20Notebook%20Solutions%20-%20Excel%2C%20SharePoint%2C%20MySQL%2C%20and%20Snowflake.md)
* [Chapter 41: MariaDB Through MaxScale and Kafka](Part%203%20-%20Open%20Mirroring/41%20-%20MariaDB%20Through%20MaxScale%20and%20Kafka.md)
* [Chapter 42: BigQuery with FabricBQSync](Part%203%20-%20Open%20Mirroring/42%20-%20BigQuery%20with%20FabricBQSync.md)
* [Chapter 43: MongoDB Through Change Streams](Part%203%20-%20Open%20Mirroring/43%20-%20MongoDB%20Through%20Change%20Streams.md)
* [Chapter 44: PostgreSQL Through Debezium and Kafka](Part%203%20-%20Open%20Mirroring/44%20-%20PostgreSQL%20Through%20Debezium%20and%20Kafka.md)
* [Chapter 45: PostgreSQL Polling with impulse_sync](Part%203%20-%20Open%20Mirroring/45%20-%20PostgreSQL%20Polling%20with%20impulse_sync.md)
* [Chapter 46: Synapse Dedicated SQL Pool Open Mirroring](Part%203%20-%20Open%20Mirroring/46%20-%20Synapse%20Dedicated%20SQL%20Pool%20Open%20Mirroring.md)
* [Chapter 47: File Publishing and Open Mirroring Test Tools](Part%203%20-%20Open%20Mirroring/47%20-%20File%20Publishing%20and%20Open%20Mirroring%20Test%20Tools.md)
* [Chapter 48: Choosing a Solution, Shared Lessons, and Further Reading](Part%203%20-%20Open%20Mirroring/48%20-%20Choosing%20a%20Solution%2C%20Shared%20Lessons%2C%20and%20Further%20Reading.md)

**Evidence and licensing:** the project chapters describe inspected public code, not certification or measured production reliability. GenericMirroring's Excel dependency and the MariaDB sample's MaxScale runtime have separate licensing restrictions. The Synapse implementation is included with the author's explicit confirmation that it is open source and customers may use it; no named licence is inferred where the inspected repository does not state one.

### Appendix

* [Appendix: Supported Sources, Mirroring Types, and Reference Tables](appendix.md)
* [Book Update History](history.md)
