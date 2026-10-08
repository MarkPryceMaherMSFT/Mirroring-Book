# Appendix: Supported Sources, Mirroring Types, and Reference Tables

This appendix summarises current source support, replication mechanisms, key terminology, and related documentation.

Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity.

## Section 1: Supported Sources and Mirroring Types

| Platform | Mirroring Type | Status | Data availability model | Chapter or guide |
|---|---|---|---|---|
| Azure SQL Database | Database mirroring | GA | Continuous replication | Chapter 11 |
| Azure SQL Managed Instance | Database mirroring | GA | Continuous replication; method depends on update policy | Chapter 12 |
| Azure Cosmos DB (NoSQL API only) | Database mirroring | GA | Continuous replication from backup | Chapter 13 |
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
| Open Mirrored Databases | Open mirroring | GA | Depends on implementation | Chapters 28–33 |
| Dremio (catalog) | Metadata mirroring | Public Preview | Metadata sync only | Chapter 26 |
| AWS Glue (catalog) | Metadata mirroring | Public Preview | Metadata sync only | Chapter 27 |
| Google Lakehouse Runtime Catalog | Metadata mirroring | Public Preview | Catalog sync and shortcuts to Google Cloud Storage | [Microsoft Learn guide](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/google-lakehouse-runtime) |

Fabric SQL Database mirroring is auto-configured when you create a Fabric SQL Database. Source availability does not imply that every optional capability is GA, and no row in this table is a latency guarantee. See the [Fabric What's New page](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new#generally-available-features) for GA announcements, each source chapter for limitations, and the [review history](history.md) for documentation conflicts.

## Section 2: Replication Mechanism Reference

| Platform | Replication Mechanism |
|---|---|
| Azure SQL Database | Change feed |
| Azure SQL Managed Instance | Change feed for Always-up-to-date or SQL Server 2025 update policy; SQL Server CDC when the instance uses the SQL Server 2022 update policy |
| Azure Cosmos DB (NoSQL API only) | Continuous Backup with a 7-day or 30-day retention window |
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
| **Continuous Backup** | The Azure Cosmos DB backup model used by Fabric mirroring to read changes from the retained backup history, not from Change Feed. |
| **Metadata mirroring** | A mirroring mode that exposes data in place through OneLake shortcuts, using catalog synchronisation or a source-specific connection integration. |
| **Change feed** | The native change stream used by selected sources, including Azure SQL Database and SQL Server 2025, to publish changes directly for Fabric mirroring. |
| **Eventhouse** | A Fabric analytics item for event and log data, useful when you need to analyse high-volume telemetry alongside mirrored operational data. |

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

**Contents:** [Table of Contents](index.md) | **Previous:** [Chapter 44: Choosing a Solution, Shared Lessons, and Further Reading](Part%203%20-%20Open%20Mirroring/chapter-44.md) | **Next:** [Book Update History](history.md)
