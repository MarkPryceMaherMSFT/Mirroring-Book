# Appendix: Supported Sources, Mirroring Types, and Reference Tables

This appendix summarises current source support, replication mechanisms, key terminology, and related documentation.

For eligible database and open mirrored replicas, Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity. Catalog shortcuts and application-managed storage have different billing boundaries.

## Section 1: Supported Sources and Mirroring Types

| Platform | Mirroring Type | Status | Data availability model | Chapter or guide |
|---|---|---|---|---|
| Azure SQL Database | Database mirroring | GA | Continuous replication | Chapter 11 |
| Azure SQL Managed Instance | Database mirroring | GA | Continuous replication; method depends on update policy | Chapter 12 |
| Azure Cosmos DB (NoSQL API only) | Database mirroring | GA | Continuous-backup initial load, then live-container increments | Chapter 13 |
| Azure Databricks (Unity Catalog) | Metadata mirroring | GA | Metadata sync only | Chapter 14 |
| Azure Monitor | Metadata mirroring | Public Preview | Connection-based access; visibility depends on the source and query path | Chapter 15 |
| Google BigQuery | Database mirroring | GA | CHANGES polling after initial export | Chapter 16 |
| Oracle | Database mirroring | GA | Log-based replication | Chapter 17 |
| Azure Database for PostgreSQL (flexible server) | Database mirroring | GA | Source-side WAL capture and publishing | Chapter 18 |
| Azure Database for MySQL (flexible server) | Database mirroring | Public Preview | Source-side binlog capture and publishing | Chapter 19 |
| SAP (via SAP Datasphere) | Database mirroring | GA | Replication flow and ADLS staging | Chapter 20 |
| SharePoint List | Database and metadata mirroring | GA | Managed list replication; shortcuts for document content | Chapter 21 |
| Snowflake | Database and metadata mirroring | GA | Streams for replicated tables; shortcuts for supported Iceberg tables | Chapter 22 |
| SQL Server 2016–2022 | Database mirroring | GA | CDC-based replication | Chapter 23 |
| SQL Server 2025 | Database mirroring | GA | Change feed | Chapter 24 |
| Fabric SQL Database | Database mirroring | GA | Automatically configured replication | Chapter 25 |
| Open Mirrored Databases | Open mirroring | GA | Depends on implementation | Chapters 32–48 |
| Dremio (catalog) | Metadata mirroring | Public Preview | Metadata sync only | Chapter 26 |
| AWS Glue (catalog) | Metadata mirroring | Public Preview | Metadata sync only | Chapter 27 |
| Google Lakehouse Runtime Catalog | Metadata mirroring | Public Preview | Catalog sync and shortcuts to Google Cloud Storage | [Chapter 30](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-30.md) |
| Dynamics 365 Business Central | Open mirroring, application-managed publisher | Prerelease; preview rollout wording conflicts | Initial/incremental source runs publish a physical OneLake replica | [Chapter 31](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-31.md) |

Fabric SQL Database mirroring is auto-configured when you create a Fabric SQL Database. Source availability does not imply that every optional capability is GA, and no row in this table is a latency guarantee. See the [Fabric What's New page](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new#generally-available-features) for GA announcements, each source chapter for limitations, and the [review history](history.md) for documentation conflicts.

**Application scope matters:** Business Central supplies a publisher for an Open Mirroring destination, not a native source in Fabric's connector picker or a fourth Mirroring type. Its prerelease status is separate from Open Mirroring's GA status.

### Related Source-Managed Integrations

These entries extend the book's coverage, not Microsoft's three-type native Mirroring taxonomy. Do not infer native Mirrored Database API, monitoring, storage allowance, or permission behavior from a similar shortcut-based experience.

| Integration | Data and ownership model | Chapter |
|---|---|---|
| Dataverse Link to Microsoft Fabric | Dataverse maintains an optimized Delta replica in Dataverse storage and manages shortcuts into a Fabric Lakehouse. Low-latency sync is GA-announced with station-based rollout; existing links need a planned migration. The replica consumes Dataverse database capacity, not the native mirrored-storage allowance. | [28: Dataverse Link and low-latency sync](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md) |
| SAP Business Data Cloud Connect for Microsoft Fabric | Announced governed data-product sharing, not the Datasphere/ADLS replication path. The original Q3 2026 target was followed by a 31 August SAP Community answer targeting end-Q1 2027; neither is GA confirmation. No Fabric-specific public setup was verified. | [29: SAP BDC Connect](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-29.md) |

### September 2026 FabCon Announcement Boundaries

The [feature summary](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabric-september-2026-feature-summary/5325825) announces BigQuery and SharePoint List mirroring GA, extended capabilities GA, Snowflake security-role replication Preview, and Cosmos DB VNet gateway support. These are different scopes: a source's GA status does not make optional role replication GA or change the Cosmos DB replication network path. The [announcement-to-chapter record](history.md#8-october-2026-fabcon-announcement-coverage) identifies where each is covered.

Its linked [OneLake FabCon companion](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabcon-and-sqlcon-barcelona-2026-what%E2%80%99s-new-in-microsoft-onelake-and-its-rapidly/5369146) also discusses catalog federation and additional integrations:

| Integration | Treatment in this book |
|---|---|
| AWS Glue and Google Lakehouse Runtime Catalog | Separate Preview walkthroughs in [Chapter 27](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md) and [Chapter 30](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-30.md). These synchronize catalog metadata and expose S3/GCS data through shortcuts; they do not copy the underlying table data into OneLake. |
| Dynamics 365 Business Central | The companion announces selected table/company data through mirroring without an explicit lifecycle label. The newer [Business Central setup guide](https://learn.microsoft.com/en-us/dynamics365/business-central/admin-set-up-fabric-mirroring) now supports a dedicated [Chapter 31 walkthrough](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-31.md), while retaining prerelease and conflicting preview-rollout wording. This is not a GA confirmation. |
| CONNECT from AVEVA | Named as new catalog federation, without an explicit lifecycle status or setup link in the announcement. Do not infer authentication, replication, or support details, or present a fabricated walkthrough. |

## Section 2: Replication Mechanism Reference

| Platform | Replication Mechanism |
|---|---|
| Azure SQL Database | Change feed |
| Azure SQL Managed Instance | Change feed for Always-up-to-date or SQL Server 2025 update policy; SQL Server CDC when the instance uses the SQL Server 2022 update policy |
| Azure Cosmos DB (NoSQL API only) | Latest continuous backup for initial backfill; live-container increments afterward. The 7-day/30-day backup window is PITR retention, not refresh cadence |
| Azure Databricks (Unity Catalog) | Unity Catalog metadata API plus OneLake shortcuts |
| Azure Monitor | Connection-based access to Log Analytics Delta Parquet storage via OneLake shortcuts; no replication pipeline |
| Google BigQuery | Google Cloud Storage initial export followed by BigQuery CHANGES TVF polling |
| Oracle | LogMiner with the source database in archive log mode |
| Azure Database for PostgreSQL (flexible server) | Source-side WAL logical replication capture, followed by Parquet publishing to OneLake |
| Azure Database for MySQL (flexible server) | Source-side binlog capture, followed by Parquet publishing to OneLake |
| SAP (via SAP Datasphere) | SAP Datasphere replication flow to ADLS Gen2; Fabric mirroring reads from ADLS |
| SharePoint List | Native Mirrored SharePoint Online List connector: OneLake shortcuts for Document Library data plus managed replication of list rows into Delta tables |
| Snowflake | Snowflake Streams polling, using a hybrid pattern |
| SQL Server 2016–2022 | SQL Server CDC via an on-premises data gateway or VNet data gateway |
| SQL Server 2025 | Change feed; requires Azure Arc and an on-premises or VNet data gateway |
| Fabric SQL Database | Auto-configured built-in mirroring |
| Open Mirrored Databases | Developer-defined |
| Dremio (catalog) | Dremio Iceberg REST Catalog with credential vending or personal access token authentication, plus OneLake shortcuts and Iceberg-to-Delta conversion |
| AWS Glue (catalog) | AWS Glue Iceberg REST Catalog with IAM access-key authentication, plus OneLake shortcuts to Amazon S3 and Iceberg-to-Delta conversion |
| Google Lakehouse Runtime Catalog | Iceberg REST Catalog authenticated through Google Cloud Workload Identity Federation; shortcuts read Iceberg V2 Parquet data in Google Cloud Storage |
| Dynamics 365 Business Central | Application-managed initial snapshot and change-tracked incremental runs to an open mirrored database; not business-data extraction through BC APIs |
| Dataverse Link to Fabric | Low-latency database-to-Delta sync in Dataverse-managed storage, consumed through Fabric lakehouse shortcuts; earlier profiles retain CSV-to-Delta conversion until migrated |

## Section 3: Key Terminology Reference

| Term | Definition |
|---|---|
| **Fabric Mirroring** | The Fabric feature that mirrors supported external or Fabric-managed sources into OneLake, or syncs metadata when the source uses metadata mirroring. |
| **Mirrored Database** | A Fabric item that represents a mirrored copy of a source database or a source whose metadata is mirrored into Fabric. |
| **Open Mirrored Database** | A Fabric item that expects an external application, service, or partner solution to write change data into the landing zone. |
| **Landing Zone** | The OneLake area where Open Mirroring files arrive before Fabric processes them into queryable tables. |
| **Replicator / Replication Engine** | The Fabric service that manages snapshots, incremental processing, retries, and table updates for mirrored items. |
| **Delta Table** | The queryable Delta Parquet table created in OneLake for database mirroring and Open Mirroring data. |
| **SQL Analytics Endpoint** | The read-only T-SQL query surface provisioned for database/open mirroring and supported mirrored catalogs. |
| **CDC (Change Data Capture)** | A mechanism that captures inserts, updates, and deletes from a source so Fabric can apply incremental changes. |
| **Watermark** | The position marker, such as a timestamp, token, or log sequence value, that records the last processed change. |
| **Backoff** | A source-specific reduction in polling or retry frequency during low activity, transient failures, or throttling. It is not a distinct replication status value. |
| **Snapshot** | The initial full load that establishes the starting state of mirrored tables. |
| **Incremental replication** | The ongoing processing of changes after the initial snapshot completes. |
| **_metadata.json** | The per-table Open Mirroring file that defines key columns and optional delimited-text or file-detection settings. |
| **__rowMarker__** | The final integer column in an incremental Open Mirroring file. It identifies insert, update, delete, or upsert operations. |
| **Direct Lake** | A Power BI storage mode that loads data from OneLake Delta tables on demand, without a scheduled import of the full dataset. It still uses memory and depends on model refresh and security configuration. |
| **Shortcut** | A OneLake reference that points to external or existing data without copying the files into a new location. |
| **Continuous Backup** | Required Cosmos DB backup infrastructure. Mirroring's initial backfill reads the latest backup; subsequent increments come from the live container, not Cosmos DB Change Feed. Retention controls PITR reach. |
| **Metadata mirroring** | A mirroring mode that exposes data in place through OneLake shortcuts, using catalog synchronisation or a source-specific connection integration. |
| **Source-managed analytical replica** | An analytical representation prepared and kept current by the source platform, such as Dataverse's optimized Delta tables; a Fabric shortcut can read it without owning a native Mirroring pipeline. |
| **Dataverse Link to Fabric** | The Power Apps integration that prepares Dataverse analytical data and manages its link/shortcuts into Fabric; separate from a native Mirrored Database connector and from customer-storage Azure Synapse Link exports. |
| **SAP BDC Connect for Microsoft Fabric** | The SAP Business Data Cloud integration for governed data-product sharing with Fabric, not the SAP Datasphere replication-flow/ADLS landing route in Chapter 20. |
| **Business Central Mirroring** | The Business Central application-managed publisher that sends selected company/table data to a Fabric open mirrored database; administered through the application's assisted setup and synchronization pages. |
| **Change feed** | The native change stream used by selected sources, including Azure SQL Database and SQL Server 2025, to publish changes directly for Fabric mirroring. |
| **Eventhouse** | A Fabric analytics item for event and log data, useful when you need to analyse high-volume telemetry alongside mirrored operational data. |
| **Link to Fabric** | Dataverse's managed analytics integration, provisioned as a lakehouse and shortcuts to a source-managed replica rather than a native Dataverse Mirroring connector. |
| **Low-latency sync** | Dataverse's database-to-Delta synchronization engine, removing an intermediate CSV stage. The product name is not a latency SLA or proof that an existing link was upgraded. |
| **Workload Identity Federation** | Google's token-federation path for the Google catalog connection, mapping approved Entra identities to Google IAM principals without a stored Google service-account key. |

## Section 4: Related Documentation

| Resource | URL |
|---|---|
| Mirroring overview | [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/mirroring/overview) |
| Open mirroring | [Overview and protocol](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring) |
| REST API reference | [Mirrored database REST API](https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-rest-api) |
| Monitor logs | [Operation log schema](https://learn.microsoft.com/en-us/fabric/mirroring/monitor-logs) |
| Troubleshooting | [Mirroring troubleshooting](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting) |
| Fabric Toolbox | [Open mirroring samples](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring) |
| Open mirroring partners | [Partner ecosystem](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-partners-ecosystem) |
| Product updates | [What's new in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new) |
| FabCon September 2026 announcements | [Feature summary](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/fabric-september-2026-feature-summary/5325825) and [book coverage record](history.md#8-october-2026-fabcon-announcement-coverage) |
| Google Lakehouse runtime catalog | [Overview, tutorial, and limitations](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/google-lakehouse-runtime) |
| Business Central application setup | [Business Central administration documentation](https://learn.microsoft.com/en-us/dynamics365/business-central/admin-set-up-fabric-mirroring) |
| Dataverse Fabric integration | [Overview](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/azure-synapse-link-view-in-fabric) and [current setup and upgrade guidance](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/fabric-link-to-data-platform) |
| Dataverse low-latency announcement | [Microsoft Power Platform blog](https://www.microsoft.com/en-us/power-platform/blog/2026/06/09/low-latency-sync/) |

**Contents:** [Table of Contents](index.md) | **Previous:** [Chapter 48: Choosing a Solution, Shared Lessons, and Further Reading](Part%203%20-%20Open%20Mirroring/chapter-48.md) | **Next:** [Book Update History](history.md)
