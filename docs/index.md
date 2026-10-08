# Fabric Mirroring: A Practical Guide

This guide covers the main mirroring patterns in Microsoft Fabric, source-specific setup, and open mirroring implementations. Use Part 1 for concepts and architecture, Part 2 for source guides, and Part 3 for open mirroring and partner-led integrations.

Each part has its own **chapter index** in a separate `readme.md`, listing only the chapters in that part. Every chapter links back to its part's index.

**FabCon September 2026:** the [announcement coverage record](history.md#8-october-2026-fabcon-announcement-coverage) links each relevant announcement to its chapter and distinguishes GA, Preview, Coming Soon, and unresolved documentation differences.

Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity.

[![Mirroring types and their outputs](assets/diagrams/index/diagram-01.png)](assets/diagrams/index/diagram-01.excalidraw.png)

* Database mirroring copies source data into Delta tables in OneLake.
* Metadata mirroring uses catalog or connection integrations and OneLake shortcuts to access source data in place.
* Open mirroring accepts developer-defined or partner-defined change data in a landing zone that Fabric processes.

## Table of Contents

### Part 1: Concepts and Architecture

**Part index:** [Chapters in Part 1](Part%201%20-%20Concepts%20and%20Architecture/readme.md)

* [Chapter 1: Introduction to Fabric Mirroring](Part%201%20-%20Concepts%20and%20Architecture/chapter-01.md)
* [Chapter 2: Types of Mirroring in Fabric](Part%201%20-%20Concepts%20and%20Architecture/chapter-02.md)
* [Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts](Part%201%20-%20Concepts%20and%20Architecture/chapter-03.md)
* [Chapter 4: The Anatomy of a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/chapter-04.md)
* [Chapter 5: Monitoring a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/chapter-05.md)
* [Chapter 6: Using the Fabric REST API](Part%201%20-%20Concepts%20and%20Architecture/chapter-06.md)
* [Chapter 7: Deploying a Mirrored Database Using CI/CD](Part%201%20-%20Concepts%20and%20Architecture/chapter-07.md)
* [Chapter 8: Using a Mirrored Database](Part%201%20-%20Concepts%20and%20Architecture/chapter-08.md)
* [Chapter 9: Extended Capabilities](Part%201%20-%20Concepts%20and%20Architecture/chapter-09.md)
* [Chapter 10: Billing and Capacity Management](Part%201%20-%20Concepts%20and%20Architecture/chapter-10.md)

### Part 2: Source-Specific Mirroring Guides

**Part index:** [Chapters in Part 2](Part%202-%20Source-Specific%20Mirroring%20Guides/readme.md)

**Start with the source, then its setup walkthrough.** Each guide takes you from source preparation and permissions through the Fabric connection, initial validation and operational troubleshooting. Do the source-administrator steps before opening the Fabric wizard; a successful sign-in does not prove that change capture or storage access is configured.

#### Setup and Troubleshooting Directory

Each setup link starts with the source-administrator work, not just the Fabric wizard. The common-problems sections distinguish documented behavior, public reports, and access-limited research leads.

| Source chapter | Practical setup | Common problems |
|---|---|---|
| [11: Azure SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-11.md) | [Identity, SQL grants, networking, and first replication](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-11.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-11.md#public-issues-and-common-pitfalls) |
| [12: Azure SQL Managed Instance](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-12.md) | [Choose the update-policy branch before configuring](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-12.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-12.md#public-issues-and-common-pitfalls) |
| [13: Azure Cosmos DB](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-13.md) | [Continuous backup, credentials, and network path](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-13.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-13.md#public-issues-and-common-pitfalls) |
| [14: Azure Databricks / Unity Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-14.md) | [External data access, UC grants, and separate ADLS access](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-14.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-14.md#public-issues-and-common-pitfalls) |
| [15: Azure Monitor](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-15.md) | [Workspace, source identity, and table permissions](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-15.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-15.md#public-issues-and-common-pitfalls) |
| [16: Google BigQuery](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-16.md) | [APIs, IAM, staging bucket, and source change history](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-16.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-16.md#public-issues-and-common-pitfalls) |
| [17: Oracle](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-17.md) | [Archive logs, supplemental logging, grants, and gateway](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-17.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-17.md#public-issues-and-common-pitfalls) |
| [18: PostgreSQL](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-18.md) | [Server parameters, identity, ownership, and replica identity](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-18.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-18.md#public-issues-and-common-pitfalls) |
| [19: MySQL](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-19.md) | [Binlogs, UAMI, connection grants, and table selection](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-19.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-19.md#public-issues-and-common-pitfalls) |
| [20: SAP](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-20.md) | [Extraction-route prerequisites, ADLS, and the Fabric mirror](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-20.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-20.md#public-issues-and-common-pitfalls) |
| [21: SharePoint List](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-21.md) | [Site access, native connector, and supported lists](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-21.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-21.md#public-issues-and-common-pitfalls) |
| [22: Snowflake](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-22.md) | [Runtime role, change tracking, retention, and Iceberg branch](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-22.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-22.md#public-issues-and-common-pitfalls) |
| [23: SQL Server 2016-2022](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-23.md) | [SQL Agent, CDC, setup/runtime permissions, and gateway](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-23.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-23.md#public-issues-and-common-pitfalls) |
| [24: SQL Server 2025](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-24.md) | [Azure Arc, primary identity, SQL grants, and network paths](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-24.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-24.md#public-issues-and-common-pitfalls) |
| [25: Fabric SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-25.md) | [Automatic mirroring, permissions, and both query surfaces](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-25.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-25.md#public-issues-and-common-pitfalls) |
| [26: Dremio Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-26.md) | [Catalog authorization and storage credential vending](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-26.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-26.md#public-issues-and-common-pitfalls) |
| [27: AWS Glue Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md) | [Glue/Lake Formation grants and S3 reads](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md#setup-walkthrough) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md#public-issues-and-common-pitfalls) |
| [28: Dataverse Link to Microsoft Fabric](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md) | [Source preparation, identity, tables, and linked Lakehouse](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md#setup-walkthrough) | [Troubleshooting](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md#troubleshooting) |
| [29: SAP Business Data Cloud Connect for Microsoft Fabric](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-29.md) | [Availability and readiness gates](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-29.md#readiness-checklist--not-an-executable-setup-procedure) | [Common misidentifications](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-29.md#troubleshooting-and-common-misidentifications) |
| [Supplement: Google Lakehouse Runtime Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md#supplementary-setup-google-lakehouse-runtime-catalog) | [Federation, runtime catalog, and GCS permissions](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md#supplementary-setup-google-lakehouse-runtime-catalog) | [Issues and pitfalls](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md#gcp-public-issues-and-common-pitfalls) |

> **Note:** Chapters 14, 15, 26, and 27 cover metadata mirroring. Chapters 21 and 22 also describe shortcut-backed paths alongside replicated tables. Chapters 15, 19, 26, and 27 cover Public Preview sources. The supplementary Google Lakehouse Runtime walkthrough is separate from both AWS Glue and BigQuery, with its own federation and storage permissions.

Chapters 28 and 29 cover related source-managed integrations: Dataverse maintains the Delta replica and manages its Lakehouse shortcuts, while SAP BDC Connect is an announced data-product sharing path distinct from Chapter 20's Datasphere/ADLS replication. They are full source chapters, without assuming identical native Mirroring item types or operating rules. The SAP BDC chapter explicitly records the absence of a verified Fabric-specific GA/setup confirmation.

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

**Start with the source, then choose what to own.** Chapters 30-35 explain the shared contract and engineering responsibilities. Chapters 36-45 examine actual GitHub implementations and tools; Chapter 46 compares them and keeps blog-only references separate. Open-source code is not a promise of production support, and does not make Fabric capacity, the source system, or publisher hosting free.

#### Foundations and shared implementation guidance

* [Chapter 30: What is Open Mirroring and Why It's Useful](Part%203%20-%20Open%20Mirroring/chapter-30.md)
* [Chapter 31: Setting Up Open Mirroring: Step-by-Step Configuration](Part%203%20-%20Open%20Mirroring/chapter-31.md)
* [Chapter 32: Code Samples and the Fabric Toolbox](Part%203%20-%20Open%20Mirroring/chapter-32.md)
* [Chapter 33: Use Cases and Examples](Part%203%20-%20Open%20Mirroring/chapter-33.md)
* [Chapter 34: Metadata and Change Files](Part%203%20-%20Open%20Mirroring/chapter-34.md)
* [Chapter 35: Common Issues and Troubleshooting](Part%203%20-%20Open%20Mirroring/chapter-35.md)

#### SDK and source-backed solution chapters

* [Chapter 36: The Microsoft Open Mirroring Python SDK](Part%203%20-%20Open%20Mirroring/chapter-36.md)
* [Chapter 37: GenericMirroring - A Multi-Source C# Publisher](Part%203%20-%20Open%20Mirroring/chapter-37.md)
* [Chapter 38: Toolbox Notebook Solutions - Excel, SharePoint, MySQL, and Snowflake](Part%203%20-%20Open%20Mirroring/chapter-38.md)
* [Chapter 39: MariaDB Through MaxScale and Kafka](Part%203%20-%20Open%20Mirroring/chapter-39.md)
* [Chapter 40: BigQuery with FabricBQSync](Part%203%20-%20Open%20Mirroring/chapter-40.md)
* [Chapter 41: MongoDB Through Change Streams](Part%203%20-%20Open%20Mirroring/chapter-41.md)
* [Chapter 42: PostgreSQL Through Debezium and Kafka](Part%203%20-%20Open%20Mirroring/chapter-42.md)
* [Chapter 43: PostgreSQL Polling with impulse_sync](Part%203%20-%20Open%20Mirroring/chapter-43.md)
* [Chapter 44: Synapse Dedicated SQL Pool Open Mirroring](Part%203%20-%20Open%20Mirroring/chapter-44.md)
* [Chapter 45: File Publishing and Open Mirroring Test Tools](Part%203%20-%20Open%20Mirroring/chapter-45.md)
* [Chapter 46: Choosing a Solution, Shared Lessons, and Further Reading](Part%203%20-%20Open%20Mirroring/chapter-46.md)

**Evidence and licensing:** the project chapters describe inspected public code, not certification or measured production reliability. GenericMirroring's Excel dependency and the MariaDB sample's MaxScale runtime have separate licensing restrictions. The Synapse implementation is included with the author's explicit confirmation that it is open source and customers may use it; no named licence is inferred where the inspected repository does not state one.

### Appendix

* [Appendix: Supported Sources, Mirroring Types, and Reference Tables](appendix.md)
* [Book Update History](history.md)
