# Appendix: Supported Sources, Mirroring Types, and Reference Tables

This appendix summarises current source support, replication mechanisms, key terminology, and related documentation.

Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity.

## Section 1: Supported Sources and Mirroring Types

| Platform | Mirroring Type | Status | Near-Real-Time Replication | Chapter |
|---|---|---|---|---|
| Azure SQL Database | Database mirroring | GA | Yes | Chapter 11 |
| Azure SQL Managed Instance | Database mirroring | GA | Yes | Chapter 12 |
| Azure Cosmos DB (NoSQL API only) | Database mirroring | GA | Yes | Chapter 13 |
| Azure Databricks (Unity Catalog) | Metadata mirroring | GA | Metadata sync only | Chapter 14 |
| Google BigQuery | Database mirroring | Public Preview | Yes | Chapter 15 |
| Oracle | Database mirroring | GA | Yes | Chapter 16 |
| Azure Database for PostgreSQL (flexible server) | Database mirroring | GA | Yes | Chapter 17 |
| Azure Database for MySQL (flexible server) | Database mirroring | Public Preview | Yes | Chapter 18 |
| SAP (via SAP Datasphere) | Database mirroring | GA | Yes | Chapter 19 |
| SharePoint List | Database mirroring | Public Preview | Yes | Chapter 20 |
| Snowflake | Database mirroring | GA | Yes | Chapter 21 |
| SQL Server 2016–2022 | Database mirroring | GA | Yes | Chapter 22 |
| SQL Server 2025 | Database mirroring | GA | Yes | Chapter 23 |
| Fabric SQL Database | Database mirroring | GA | Yes | Chapter 24 |
| Open Mirrored Databases | Open mirroring | GA | Depends on implementation | Chapters 26–31 |
| Dremio (catalog) | Metadata mirroring | Public Preview | Metadata sync only | Chapter 25 |

Note: Fabric SQL Database mirroring is auto-configured when you create a Fabric SQL Database. SharePoint List mirroring is a native Preview connector: Document Library data is exposed through OneLake shortcuts, and list row data is replicated into Delta tables.

## Section 2: Replication Mechanism Reference

| Platform | Replication Mechanism |
|---|---|
| Azure SQL Database | Fabric mirroring change feed |
| Azure SQL Managed Instance | Fabric mirroring change feed for Always-up-to-date or SQL Server 2025 update policy; SQL Server CDC when the instance uses the SQL Server 2022 update policy |
| Azure Cosmos DB (NoSQL API only) | Continuous Backup with a 7-day or 30-day retention window |
| Azure Databricks (Unity Catalog) | Unity Catalog metadata API plus OneLake shortcuts |
| Google BigQuery | Google Cloud Storage initial export followed by BigQuery CHANGES TVF polling |
| Oracle | LogMiner with the source database in archive log mode |
| Azure Database for PostgreSQL (flexible server) | PostgreSQL WAL logical replication |
| Azure Database for MySQL (flexible server) | MySQL binary log, or binlog |
| SAP (via SAP Datasphere) | SAP Datasphere replication flow to ADLS Gen2; Fabric mirroring reads from ADLS |
| SharePoint List | Native Mirrored SharePoint Online List connector: OneLake shortcuts for Document Library data plus managed replication of list rows into Delta tables |
| Snowflake | Snowflake Streams polling, using a hybrid pattern |
| SQL Server 2016–2022 | SQL Server CDC via an on-premises data gateway or VNet data gateway |
| SQL Server 2025 | Fabric mirroring change feed plus Azure Arc and an on-premises or VNet data gateway |
| Fabric SQL Database | Auto-configured built-in mirroring |
| Open Mirrored Databases | Developer-defined |
| Dremio (catalog) | Dremio Iceberg REST Catalog with credential vending or personal access token authentication, plus OneLake shortcuts and Iceberg-to-Delta conversion |

## Section 3: Key Terminology Reference

| Term | Definition |
|---|---|
| **Fabric Mirroring** | The Fabric feature that mirrors supported external or Fabric-managed sources into OneLake, or syncs metadata when the source uses metadata mirroring. |
| **Mirrored Database** | A Fabric item that represents a mirrored copy of a source database or a source whose metadata is mirrored into Fabric. |
| **Open Mirrored Database** | A Fabric item that expects an external application, service, or partner solution to write change data into the landing zone. |
| **Landing Zone** | The OneLake area where Open Mirroring files arrive before Fabric processes them into queryable tables. |
| **Replicator / Replication Engine** | The Fabric service that manages snapshots, incremental processing, retries, and table updates for mirrored items. |
| **Delta Table** | The queryable Delta Parquet table created in OneLake for database mirroring and Open Mirroring data. |
| **SQL Analytics Endpoint** | The automatically provided T-SQL endpoint for querying mirrored tables in Fabric. |
| **CDC (Change Data Capture)** | A mechanism that captures inserts, updates, and deletes from a source so Fabric can apply incremental changes. |
| **Watermark** | The position marker, such as a timestamp, token, or log sequence value, that records the last processed change. |
| **Backoff** | Retry behaviour where Fabric waits before trying again after a transient failure or throttling event. It is not a distinct replication status value; it occurs while a mirror shows Running or Running with warning. |
| **Snapshot** | The initial full load that establishes the starting state of mirrored tables. |
| **Incremental replication** | The ongoing processing of changes after the initial snapshot completes. |
| **_metadata.json** | The per-table Open Mirroring file that defines key columns and optional delimited-text or file-detection settings. |
| **__rowMarker__** | The final integer column in an incremental Open Mirroring file. It identifies insert, update, delete, or upsert operations. |
| **DirectLake** | A Power BI storage mode that reads OneLake data directly, without importing it into a separate semantic model cache first. |
| **Shortcut** | A OneLake reference that points to external or existing data without copying the files into a new location. |
| **Continuous Backup** | The Azure Cosmos DB backup model used by Fabric mirroring to read changes from the retained backup history, not from Change Feed. |
| **Metadata mirroring** | A mirroring mode where Fabric synchronises catalog metadata and exposes source data through OneLake shortcuts instead of copying the data into Delta tables. |
| **Fabric mirroring change feed** | The native change stream used by selected sources, including Azure SQL Database and SQL Server 2025, to publish changes directly for Fabric mirroring. |
| **Eventhouse** | A Fabric analytics item for event and log data, useful when you need to analyse high-volume telemetry alongside mirrored operational data. |

## Section 4: Related Documentation

| Resource | URL |
|---|---|
| Mirroring overview | `https://learn.microsoft.com/en-us/fabric/mirroring/overview` |
| Open mirroring | `https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring` |
| REST API reference | `https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-rest-api` |
| Monitor logs | `https://learn.microsoft.com/en-us/fabric/mirroring/monitor-logs` |
| Troubleshooting | `https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting` |
| Fabric samples on GitHub | `https://github.com/microsoft/fabric-samples` |
| Open mirroring partners | `https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-partners-ecosystem` |

**Contents:** [Table of Contents](index.md) | **Previous:** Chapter 31: Common Issues and Troubleshooting | **Next:** Book Update History
