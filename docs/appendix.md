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
| SQL Server 2025 | Change feed and Azure Arc; documented walkthrough uses an on-premises or VNet data gateway, with public-route wording differences recorded in Chapter 24 |
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

## Section 5: Data Gateway Reference

**Review cutoff: 4 November 2026.** Gateway support belongs to a **specific connector and workload**, not simply a database product or a network location. A Power Query, pipeline, Copy Job, DirectQuery, or standalone shortcut connector does not establish support for the same source in native Mirroring. VNet data gateways and standard-mode on-premises data gateways are also different from Azure VPN Gateway, managed private endpoints, and Azure Data Factory's self-hosted integration runtime.

### Gateway Support by Source

**Legend:** **Supported** means the source's Mirroring procedure explicitly provides that gateway option. **Unsupported** is an explicit product exclusion. **Not documented** means no native setup/support contract was verified; it is not a blanket claim that the gateway can never reach that product. **Not part of this flow** describes a service/application-managed integration without a gateway step. Public reachability does not mean anonymous access.

| Source chapter | VNet data gateway | Standard-mode on-premises gateway | Connection and replication boundary |
|---|---|---|---|
| [11: Azure SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-11.md) | Supported; private access needs a gateway, approved public path can use None | Supported on the same connection-routing basis | Gateway handles SQL connection/setup/metadata. Source-engine snapshot/change-feed publication to OneLake and publishing-identity authorization are separate. |
| [12: SQL MI, Always-up-to-date/2025 policy](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-12.md) | Supported for private access; approved public path can use None | Supported on the same connection-routing basis | Gateway connection is separate from the change-feed publisher's SAMI-authenticated OneLake uploads. Do not copy logical-server Resource Instance Rules to MI without source-specific guidance. |
| [12: SQL MI, 2022 policy](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-12.md) | Supported; a gateway is **mandatory**, including public sources | Supported; a gateway is **mandatory**, including public sources | Snapshot and ongoing CDC reads traverse the gateway. This is not the newer MI change-feed publishing path. |
| [13: Azure Cosmos DB](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-13.md) | Supported for restricted-account connection creation/testing; not needed for public setup | Not documented for native Mirroring | OAuth, trusted-workspace ACL bypass, private DNS/subnet, same-region workspace, and REST item creation. Later implementation docs place ongoing replication in Fabric Replicator, **not through either gateway or source private endpoint**. |
| [14: Databricks Unity Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-14.md) | Supported for private workspace/catalog access; gateway region matches Databricks workspace | **Unsupported** | Precreate the gateway-backed Azure Databricks workspace connection. Catalog/API connectivity does not prove access to ADLS table files; trusted workspace access and storage restrictions are separate. |
| [15: Azure Monitor](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-15.md) | Not documented for native catalog mirroring | Not documented for native catalog mirroring | Service-managed access to existing Log Analytics Delta data. Export rules, agent DCR destinations, and generic query connectors are different integrations. |
| [16: Google BigQuery](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-16.md) | Supported, optional | Supported, optional; historical feature floor **3000.286.6** | Source authentication, initial export, GCS staging, and `CHANGES` polling remain required. A gateway does not override this mirror's Fabric inbound public-access restrictions. |
| [17: Oracle](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-17.md) | **Unsupported** | **Required**, including public/cloud Oracle; historical feature floor **3000.282.5** | The gateway runs **Oracle Mirror Publisher**, with Basic source authentication and 64-bit OCMT. It is not merely a discovery proxy; do not infer publisher HA from general gateway clustering. |
| [18: Azure PostgreSQL](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-18.md) | Supported for private/VNet-integrated source access; public cloud connection also documented | Not documented for native Mirroring | Source-side `azure_cdc` capture/publishing and SAMI-authenticated OneLake uploads are separate from the gateway connection. No numeric gateway minimum is published in the source procedure. |
| [19: Azure MySQL](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-19.md) | **Unsupported** | Not documented for native Mirroring | Tutorial selects **Gateway: None**. The source publisher uses UAMI for OneLake uploads. Stated source Private Link support does not supply a complete private provisioning recipe or imply VNet gateway support. |
| [20: SAP via Datasphere](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-20.md) | Not documented for this flow | Not part of this flow | SAP extraction/Datasphere → ADLS staging → shortcut → Fabric replication. SAP Cloud Connector is a separate upstream component, not a Fabric data gateway. |
| [21: SharePoint Online List](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-21.md) | Native option not documented | Native option not documented | Cloud list rows replicate and Document Library content uses shortcuts. An on-premises SharePoint connector elsewhere does not establish native SharePoint Server mirroring. |
| [22: Snowflake](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-22.md) | Supported, optional | Supported, optional | Query/extraction connectivity, OneLake unload/storage-integration authorization, and Iceberg shortcut reads are separate. Restricted inline unload needs storage integration with either gateway. |
| [23: SQL Server 2016-2022](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-23.md) | Supported; required for private access; walkthrough uses gateway | Supported on the same basis | Gateway carries snapshot/CDC reads. General public-endpoint prerequisite wording and the explicit walkthrough differ; do not promise a gatewayless public deployment from that wording alone. No Arc/change-feed prerequisite for this branch. |
| [24: SQL Server 2025](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-24.md) | Supported; required for private access; walkthrough uses gateway | Supported on the same basis | Gateway handles the SQL connection; engine snapshot/change-feed publishing uses the separate **Arc-backed SAMI** path. Public-endpoint prerequisite/walkthrough wording remains inconsistent. |
| [25: Fabric SQL Database](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-25.md) | Not part of this flow | Not part of this flow | Automatic internal replication, not an external source connection. Client Private Link has its own setup/preview ordering constraints. |
| [26: Dremio Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-26.md) | Not documented; public path required | Not documented; public path required | Both the Dremio project and all backing table storage must be publicly reachable with authentication. Gateway connectivity mentioned as future work is not released support. |
| [27: AWS Glue Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-27.md) | Not documented; public path required | Not documented; public path required | Public Glue REST and all S3 storage locations. A standalone S3 shortcut gateway is not a gateway for this mirrored catalog. |
| [28: Dataverse Link to Fabric](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-28.md) | Not part of direct Link; no gateway workaround documented | Not part of direct Link | Application-managed Delta replica and Lakehouse shortcuts. The Link FAQ requires inbound all-networks access; an approved outbound environment-URL rule is a different control. |
| [29: SAP BDC Connect](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-29.md) | No verified released Fabric recipe | No verified released Fabric recipe | Announced governed sharing; do not import another partner's transport, gateway, or authentication configuration. |
| [30: Google Lakehouse Runtime Catalog](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-30.md) | Not documented; public path required | Not documented; public path required | Public REST catalog and all GCS locations. A standalone GCS shortcut's gateway is not native mirrored-catalog support. |
| [31: Business Central Mirroring](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-31.md) | Not part of documented publisher flow | Not part of documented publisher flow | Business Central online's same-tenant, prerelease Open Mirroring publisher. No publisher-specific private-endpoint/gateway recipe is established by its current setup. |
| [32-48: Open Mirroring implementations](Part%203%20-%20Open%20Mirroring/chapter-33.md#publisher-networking-and-data-gateways) | No generic gateway requirement | No generic gateway requirement | The publisher owns source reachability and OneLake upload connectivity; a particular partner/application can add its own requirements. |

For **database mirrors**, the current [tenant](https://learn.microsoft.com/en-us/fabric/security/security-private-links-overview#mirrored-database) and [workspace](https://learn.microsoft.com/en-us/fabric/security/security-workspace-level-private-links-support#mirrored-database-support) inbound-protection guides list Open Mirroring, Cosmos DB, SQL Managed Instance, SAP, SharePoint List, and SQL Server 2025. Other database-mirroring types pause/cannot start when the applicable public inbound access is denied. A source-private gateway path does not override that supported-item list. Metadata catalogs, Fabric SQL, and application links retain their own additional restrictions; in particular, Iceberg virtualization has separate Private Link exclusions.

**Source data traffic is not implied by gateway selection.** Cosmos DB's later implementation guidance contradicts its September announcement on that point; [Chapter 13](Part%202-%20Source-Specific%20Mirroring%20Guides/chapter-13.md#network-and-connectivity) records both. Historical source minimum gateway builds in this table must be combined with the **current supported/latest release baseline** below, not installed as an ongoing production recommendation.

### Shared Gateway Setup and Operations

#### Standard-mode on-premises data gateways

Use the [standard gateway](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-install) for supported Fabric integrations. **Personal mode is Power BI-only**, not a native Mirroring gateway. The software can run on a suitable Azure VM as well as an on-premises machine; its name does not require the database or gateway host to be physically on premises.

Install on a supported, always-available Windows host, not a laptop, domain controller, Server Core installation, or Windows container. Protect the recovery key and use a named operational owner. Follow the source chapter for drivers, publisher components, credentials, and any source-specific high-availability exclusions. A general gateway cluster capability does not automatically make every publisher fail over safely.

The gateway initiates **outbound** cloud connections using Azure Relay; it does not require an internet-facing inbound listener. It still needs a permitted connection to the source. Validate both routes using the [communication/endpoints guide and network ports test](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-communication). HTTPS mode concerns the Azure Relay leg, not a promise that SQL/Oracle traffic or a source's separate OneLake publishing leg uses only port 443. Do not copy Dataflow Gen2 destination allowlists or disable certificate checks as a universal Mirroring fix.

Configure an approved [proxy](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-proxy) for the actual configurator, Windows service, and relevant worker processes, not only the interactive user's browser. A successful laptop login or gateway status test does not prove that the publisher's service identity and direct destination path work.

#### Managed VNet data gateways

The [VNet gateway overview](https://learn.microsoft.com/en-us/data-integration/vnet/overview) includes Fabric Mirroring as a workload, **not all Mirroring sources**. The gateway is Microsoft-managed compute injected into an Azure subnet; users do not install the Windows gateway on its nodes. All F SKUs are eligible, with **F8+ recommended rather than required**. The gateway's P/A4+ and trial eligibility is separate from the Fabric item's own capacity requirements.

Follow [gateway creation](https://learn.microsoft.com/en-us/data-integration/vnet/create-data-gateways): register `Microsoft.PowerPlatform`, obtain `Microsoft.Network/virtualNetworks/subnets/join/action`, and dedicate an IPv4 subnet delegated to `Microsoft.PowerPlatform/vnetaccesslinks`. Do not share it with unrelated services, add IPv6 address space, or use reserved subnet names such as `GatewaySubnet`. Size for reserved addresses, gateway members, and growth; apply a source-specific minimum such as Cosmos DB's `/27` where documented rather than treating that minimum as universal.

The running gateway is in the **VNet's region**; its management metadata is stored in the **tenant home region**. Check gateway-region availability, same-tenant requirements, and the particular source's region rule separately. Private DNS, routes/peering, NSGs, OAuth/Entra egress, and certificate validation must work from the delegated subnet. A private endpoint is not a substitute for authentication egress. Where required by the source procedure, provide explicit NAT/outbound connectivity rather than relying on legacy default outbound access. Storage-dependent paths may additionally require `Microsoft.Storage`; catalog discovery success does not prove that data files are readable.

The [current management guide](https://learn.microsoft.com/en-us/data-integration/vnet/manage-data-gateways#certificate--proxy-support) supports enterprise certificates and proxies; upload **individual certificate files**, not a multi-certificate CA bundle. This is not permission to change every source's authentication method to client certificates. The [VNet role reference](https://learn.microsoft.com/en-us/data-integration/vnet/manage-vnet-security-roles) documents gateway Admin/Connection creator roles separately from connection Owner/User roles; older on-premises documentation that says VNet supports only Admin is not the current VNet role specification.

The [VNet HA guide](https://learn.microsoft.com/en-us/data-integration/vnet/high-availability-load-balancing) documents 1-9 members and an inactivity timeout from 30 minutes to 24 hours, with no always-on setting. Its refresh/DirectQuery/pipeline concurrency figures are **not a published Mirroring table or stream limit**. Removing a gateway is not a routine mirror repair. Resource cleanup is asynchronous; the newer [management guide](https://learn.microsoft.com/en-us/data-integration/vnet/manage-data-gateways#remove-vnet-data-gateways) warns that final subnet/VNet deletion can take 48-72 hours, despite shorter estimates in older FAQs.

For either gateway, keep **gateway administration**, **connection use/ownership**, **workspace/item permissions**, and **source authentication** separate. Fabric inbound Private Link, source private endpoints, outbound connection rules, and source-to-OneLake publishing are independent controls. Use the source's supported topology instead of opening a firewall or disabling a tenant policy merely to make a connection test pass.

### Gateway Releases and Operational Baseline

The [supported monthly-release index](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-monthly-updates), reviewed **4 November 2026**, lists these six on-premises release families:

These are **Windows on-premises gateway releases**, not packages to install on the Microsoft-managed VNet gateway. VNet feature availability and connector/workload support must be checked in their own documentation.

| Release | Version family |
|---|---|
| October 2026 | **3000.338** |
| September 2026 | 3000.334 |
| August 2026 | 3000.330 |
| July 2026 | 3000.326 |
| June 2026 | 3000.322 |
| May 2026 | 3000.318 |

Microsoft supports the **last six releases** and recommends the **latest** for Fabric. Install the latest available patch, not just the first build in a listed family. An older version printed in a source tutorial is a compatibility floor, **not a currently supported production baseline**. No November gateway release is established by this index at the review cutoff.

The [identity sign-in update](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-update-signin) completed enforcement by **31 August 2026**. May 2026 and later releases contain the fix; unpatched January-April builds can fail interactive installation, recovery, update, and administration sign-in even if existing background operation still works. Do not retain an obsolete source-minimum build because an existing connection happens to run.

Use the [update procedure](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-update): drain and update one cluster member at a time, then return all members to the same current version. Thirty minutes is a typical drain estimate, not a guarantee for long-running work. The UI/PowerShell update mechanism requires a **November 2025 or later installed baseline**, gateway Admin, and the documented free disk space. That updater baseline is not a substitute for the current six-release support window.

The [June release](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/on-premises-data-gateway-june-2026-release/5233421) documents WAM sign-in, dependency updates, and consent-controlled diagnostics; the [July release](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/on-premises-data-gateway-july-2026-release/5356257) describes platform/security/authentication/diagnostic improvements and Desktop compatibility. Those are not announcements that every connector or Dataflow Gen2 diagnostic feature is available in native Mirroring.

The [October release](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/on-premises-data-gateway-october-2026-release/5376557) describes **3000.338 / Power Query Engine 2.159.229**, installer/configurator accessibility, preservation of existing Power Query settings during connection tests, improved service-start diagnostics, and October Power BI Desktop compatibility. Its tenant-wide Admin Connections API discussion is a separate governance capability, not a feature unlocked simply by upgrading the Windows binary. The captured article body announces **no native Mirroring source expansion, authentication change, or source-specific fix/minimum build**.

**Release access limitation:** the May, August, and September announcement bodies returned HTTP 403 during this review. October's body was readable once, but subsequent requests also returned 403; no exact blog publication day was captured. All six version families were verified directly in Microsoft Learn. Search-index summaries were not used to invent new Mirroring support or defect fixes. Follow the release links in the supported index when reviewing an upgrade.

For VNet gateways, the [certificate/proxy GA announcement](https://blog.fabric.microsoft.com/en-us/blog/certificate-and-proxy-support-for-vnet-data-gateway-generally-available) and the [3 November management-page update](https://learn.microsoft.com/en-us/data-integration/vnet/manage-data-gateways) establish the supported enterprise configuration. The [modern evaluation engine](https://learn.microsoft.com/en-us/data-integration/vnet/modern-evaluation-engine) remains an opt-in **Preview** for documented refresh/DirectQuery scenarios; its benchmarks are not native Mirroring replication-throughput guarantees.

**Cost and diagnostics:** VNet uptime consumes the linked capacity at **4 CUs per active member**, including applicable idle timeout after a test/query; self-hosted on-premises gateways instead require separately budgeted machines and maintenance. See [Chapter 10](Part%201%20-%20Concepts%20and%20Architecture/chapter-10.md#vnet-gateway-uptime-is-not-replication-time). Gateway logs and connection tests do not replace item/table replication status or source capture diagnostics; see [Chapter 5](Part%201%20-%20Concepts%20and%20Architecture/chapter-05.md#gateway-diagnostics-are-a-separate-layer).

**Contents:** [Table of Contents](index.md) | **Previous:** [Chapter 48: Choosing a Solution, Shared Lessons, and Further Reading](Part%203%20-%20Open%20Mirroring/chapter-48.md) | **Next:** [Book Update History](history.md)
