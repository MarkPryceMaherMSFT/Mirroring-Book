# Book Update History

This page records documentation reviews, product-status changes, and material corrections. The chapters focus on how Fabric Mirroring works and how to use it. For the current source matrix, see the [appendix](appendix.md).

Microsoft Learn remains the authoritative source for current product availability, limits, and licensing.

## 8 October 2026 (All three parts published)

Published all 44 chapters to the [public edition](https://github.com/MarkPryceMaherMSFT/Mirroring-Book), extending the previously published Part 1 with Part 2: Source-Specific Mirroring Guides and Part 3: Open Mirroring. Included the practical setup walkthroughs, attributed Microsoft documentation images, public troubleshooting references, dedicated SDK and open-source solution chapters, and source-to-project comparisons.

Published the separate `readme.md` chapter indexes for all three parts, the complete table of contents, appendix, diagrams, and update history. Updated the README to show that all three parts are published and direct issue reports to the public repository. The source repository and public edition contain the same published book content; the public repository retains its own Git history.

## 8 October 2026 (Practical source setup and part navigation)

Expanded the existing Part 2 setup walkthroughs for all 17 source chapters, including the separate Google Lakehouse Runtime Catalog walkthrough in Chapter 27. The guides cover source-administrator prerequisites, source changes and permissions, identity and networking, Fabric configuration, initial-data and supported-change checks, and operational handoff. Added a [setup and troubleshooting directory](index.md#setup-and-troubleshooting-directory) with direct links for each source.

Filled source-specific gaps rather than applying one generic recipe: examples include SQL deployment and CDC prerequisites, Oracle client configuration, BigQuery change-history checks, PostgreSQL ownership and WAL considerations, MySQL session/global settings, SAP extraction-route distinctions, SharePoint field-name pitfalls, and catalog-versus-storage authorization. Conflicting public guidance remains explicitly identified, including Cosmos DB TTL behavior and Fabric SQL full-text support.

Retained the attributed Microsoft documentation screenshots and added the Databricks private-connection selection screenshot. Captions identify the source article, original image, Microsoft credit, CC BY 4.0 licence, and alteration status. Shared monitoring images remain identified as shared examples rather than source-specific wizard screens.

Refreshed public issue research starting with Reddit and using readable Microsoft Fabric Community reports. Direct Reddit access was blocked during this review; inaccessible threads are labelled as unverified or search-index leads, without claiming their replies or resolutions were confirmed. Community incidents are dated and distinguished from supported product requirements.

Added a separate `readme.md` chapter index in each of the three part folders, listing only that part's chapters. Replaced the initial inline collapsible indexes with a link to the relevant part index in all 44 chapters. The root README and full table of contents also link to these part indexes; existing full-book contents and previous/next links remain available.

## 8 October 2026 (Open-source Open Mirroring solutions)

Expanded Part 3 with Chapters 34-44: a dedicated Microsoft Python SDK walkthrough; public GitHub implementations covering SQL Server Change Tracking, files, SharePoint, MySQL, Snowflake, MariaDB, BigQuery, MongoDB, PostgreSQL and Synapse dedicated SQL pool; practical test tools; and a source-to-project comparison.

Preserved Chapters 28-33 as the shared configuration, protocol and recovery guidance. Added cross-links and continuous chapter navigation rather than duplicating that material in every project chapter.

The new chapters distinguish publisher licensing from dependency/runtime licensing and operating costs, educational samples from production support, and source capture from landing-zone delivery. The Synapse project is included with the author's confirmation of open-source/customer use; no specific licence name is invented. Implementation observations come from public source review, not a claim that deployments or failure scenarios were executed.

Added a separate source-to-blog table for articles where matching public producer code was not found in the research. Source-backed articles remain with their relevant projects. See [Chapter 44](Part%203%20-%20Open%20Mirroring/chapter-44.md).

## 6 October 2026 (Whole-book public documentation review)

Reviewed Chapters 1-33, the contents, appendix, and README against public Microsoft Learn documentation. The entries below record corrections to the book, not a claim that every referenced feature was released on this date.

### Source coverage and availability

- Updated Google BigQuery to GA following the August announcement and SharePoint List to GA following the September announcement. Corrected SharePoint and Snowflake to distinguish replicated tables from shortcut-backed content. The generic mirroring overview still contains some older preview labels. Sources: [Fabric release updates](https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new#generally-available-features), [BigQuery](https://learn.microsoft.com/en-us/fabric/mirroring/google-bigquery), [SharePoint List](https://learn.microsoft.com/en-us/fabric/mirroring/sharepoint-list), and [Snowflake](https://learn.microsoft.com/en-us/fabric/mirroring/snowflake).
- Added Google Lakehouse Runtime Catalog mirroring to the metadata-mirroring overview, source reference tables, and source-group diagram. It is a separate Preview integration from BigQuery database replication: Iceberg V2 tables remain in Google Cloud Storage, and Microsoft Entra identities authenticate through Google Cloud Workload Identity Federation. Recorded the 500-table limit and public-network requirement. Sources: [overview](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/google-lakehouse-runtime), [tutorial](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/google-lakehouse-runtime-tutorial), and [limitations](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/google-lakehouse-runtime-limitations).
- Preserved Azure Monitor and AWS Glue as metadata mirroring. Clarified Azure Monitor's Eventhouse/KQL and shortcut access paths rather than promising an automatically created SQL analytics endpoint for every metadata connector.

### Architecture, operations, and costs

- Corrected PostgreSQL and MySQL from generic Fabric-side WAL/binlog consumers to source-side capture and publication. PostgreSQL uses its Azure integration and system-assigned identity; MySQL's documented setup uses a user-assigned identity. Corrected Oracle's gateway component to the Oracle Mirror Publisher. Sources: [PostgreSQL architecture](https://learn.microsoft.com/en-us/azure/postgresql/integration/concepts-fabric-mirroring), [MySQL architecture](https://learn.microsoft.com/en-us/azure/mysql/integration/fabric-mirroring-mysql), and [Oracle limitations](https://learn.microsoft.com/en-us/fabric/mirroring/oracle-limitations).
- Removed blanket claims that push-based mirroring never needs a gateway, polling always needs one, shortcuts have zero latency, or splitting tables between mirrors doubles throughput. Kept push/pull/shortcuts explicitly identified as this book's explanatory model, not a Microsoft product taxonomy.
- Distinguished source firewall and gateway connectivity, Fabric private link, and workspace outbound access protection. Documented source-specific data connection rules rather than inferring outbound-protection support from gateway support. Source: [Outbound access protection for mirrored databases](https://learn.microsoft.com/en-us/fabric/security/workspace-outbound-access-protection-mirrored-databases).
- Corrected OneLake security role scope, default membership, and SQL analytics endpoint identity modes. A user's OneLake roles are not automatically enforced through a delegated-owner query path. Added item-sharing permission guidance and separated source publishing identities from API callers. Sources: [OneLake security model](https://learn.microsoft.com/en-us/fabric/onelake/security/data-access-control-model), [SQL endpoint access modes](https://learn.microsoft.com/en-us/fabric/onelake/security/sql-analytics-endpoint-onelake-security), and [mirrored-item sharing](https://learn.microsoft.com/en-us/fabric/mirroring/share-and-manage-permissions).
- Clarified managed V-Ordered layout, automatic VACUUM, and configurable Delta retention. The retention threshold is not a published maintenance schedule. Open Mirroring's seven-day processed landing-file cleanup is a separate mechanism; this corrects the conflation in the August retention edit. Fabric does not maintain source files behind metadata shortcuts. Sources: [Optimise mirrored data](https://learn.microsoft.com/en-us/fabric/fundamentals/table-maintenance-optimization#optimize-mirrored-data), [Delta retention](https://learn.microsoft.com/en-us/fabric/mirroring/overview#retention-for-mirrored-data), and [landing-zone format](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-landing-zone-format).
- Corrected REST definitions, permissions, asynchronous responses, pagination, deployment behaviour, Git file names, and Terraform examples. Deployment does not replace an explicit start and status check. Sources: [Mirroring REST API](https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-rest-api), [CI/CD](https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-cicd), and [Terraform resource](https://github.com/microsoft/terraform-provider-fabric/blob/main/docs/resources/mirrored_database.md).
- Distinguished the Replication Status experience from the documentation's "Monitor replication" section rather than claiming an unverified tab rename. Corrected monitoring schema and metric interpretations, including processed operations versus destination row counts. Sources: [monitoring](https://learn.microsoft.com/en-us/fabric/mirroring/monitor) and [operation logs](https://learn.microsoft.com/en-us/fabric/mirroring/monitor-logs).
- Corrected Direct Lake variants, SQL metadata-sync dependencies, same-workspace SQL joins, and source-specific string limits. Replaced Azure Data Studio recommendations with supported SQL tools. Sources: [Direct Lake](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview), [metadata sync](https://learn.microsoft.com/en-us/fabric/data-engineering/sql-analytics-endpoint-metadata-sync), and [Azure Data Studio retirement](https://learn.microsoft.com/en-us/sql/tools/whats-happening-azure-data-studio).
- Corrected VNet gateway costs: they are billed against the linked Fabric or Premium capacity, separately from free core mirroring replication. Qualified replica-storage allowances and removed an unverified separate setup-charge claim. Sources: [VNet gateway business model](https://learn.microsoft.com/en-us/data-integration/vnet/data-gateway-business-model) and [Fabric pricing](https://azure.microsoft.com/en-us/pricing/details/microsoft-fabric/).

### Open Mirroring correctness

- Updated Part 3 against the September publication and recovery guidance: immutable final paths, durable source-range-to-file assignments, conditional publication, timestamp-detection ordering limits, and targeted table rebuilds with waits for removal and recreation. Stopping and restarting an entire Open Mirrored Database is not a single-table recovery operation. Source: [Open Mirroring best practices](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices).
- Corrected row-marker and delete semantics, metadata versus item definitions, delimited-text requirements, supported schema changes, and the difference between source checkpoints and file sequences. Hardened illustrative upload, database-polling, and Excel-conversion examples without presenting them as complete production connectors. Source: [Landing-zone format](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-landing-zone-format).
- Distinguished Microsoft-published SDK and Toolbox examples from product support or production-reliability guarantees. Source: [Fabric Toolbox support statement](https://github.com/microsoft/fabric-toolbox#support).

### Documentation conflicts and unresolved limits

The book records these differences rather than inventing a common answer. Use the source-specific implementation guide and confirm ambiguous requirements before deployment.

| Topic | Public documentation discrepancy and treatment |
|---|---|
| Extended capabilities | September release notes announce change feeds and source-view mirroring as GA, while the [extended-capabilities overview](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities) and [Views page](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-views) retain Preview labels. The book reports the conflict; paid scope and Snowflake-only view support remain separately stated. |
| Eventstreams mirrored change feed | Release summaries contain both Preview and GA references; the [connector guide](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-source-mirrored-database-change-feed) still documents Preview, All tables only, and no DeltaFlow. Those restrictions were not removed based on a summary announcement. |
| Snowflake security-role mirroring | A Preview announcement exists, but linked guidance does not establish configuration or complete policy coverage. No automatic replication of all source security policies is promised. |
| Cosmos DB private networking | The newer [private-network guide](https://learn.microsoft.com/en-us/fabric/mirroring/azure-cosmos-db-private-network) describes VNet gateway, OAuth, and REST creation; older overview and limitations text describes gatewayless ACL bypass. Chapter 13 distinguishes these paths and the conflict. |
| Azure SQL Managed Instance permissions | Tutorial granular grants and limitations-page CONTROL/db_owner requirements are not fully aligned. Chapter 12 calls out the difference rather than silently lowering required privileges. |
| BigQuery staging | Tutorial and security guidance require a staging bucket; FAQ wording denies intermediate staging. Chapter 16 follows the concrete setup requirements and notes the discrepancy. |
| PostgreSQL support boundaries | Source-service and Fabric pages differ on replica topology, types, DDL, column names, and database-count scope. Chapter 18 identifies these differences and avoids universal claims. |
| MySQL setup | Tier names, identity type, and table-selection mutability differ between FAQ and setup guidance. Chapter 19 uses the concrete UAMI setup and records the conflicting claims. |
| SQL Server gateways | General prerequisites use conditional wording, while version-specific walkthroughs prescribe a gateway. The separate SQL MI 2022-policy requirement is explicit and is not inferred from SQL Server guidance. |
| Fabric SQL start/stop | Older overview text says mirroring cannot stop, while newer [start/stop REST guidance](https://learn.microsoft.com/en-us/fabric/database/sql/start-stop-mirroring-api) supports it. Chapter 25 follows the specific API guidance. |
| Dremio authentication | The tutorial includes Organizational account authentication beyond the overview's narrower list. Catalog authentication and storage credential vending are described separately. |
| API source changes | General troubleshooting says changing the source database is unsupported; the REST guide permits specific definition changes in `Initialized` or `Stopped`. The API chapter states those conditions. |
| Capacity SKU table | Live pricing and licensing list F4096/F8192, but the book retains the author's earlier explicit F2048 ceiling. This discrepancy is not silently treated as resolved. |
| Open Mirroring specification | Marker placement, metadata omission, CSV schema evolution, and some example JSON are inconsistent within public guidance. The book uses valid JSON and conservative, explicitly documented contracts, and distinguishes examples from guarantees. |

## 8 August 2026 (Delta table retention and VACUUM accuracy in Chapters 4 and 10)

### Technical corrections

- Chapter 4's Landing Zone section previously stated that processed files are removed after a flat 7 days, with no mention of VACUUM or configurability. Replaced this with a new "Delta Table Retention and VACUUM" subsection: mirroring automatically runs Delta VACUUM to remove old, unreferenced files once they fall outside the retention window; the window is configurable from 1 to 30 days; and the default differs by creation path, 1 day for mirrored databases created through the Fabric portal after mid-June 2025, 7 days for older mirrored databases or any mirror created through the REST API without an explicit value. Added the two configuration paths: the Settings > Delta table management tab in the portal, or the `retentionInDays` property on the REST API. Sources: [Retention for mirrored data](https://learn.microsoft.com/en-us/fabric/mirroring/overview#retention-for-mirrored-data) and [mirrored database REST API](https://learn.microsoft.com/en-us/fabric/mirroring/mirrored-database-rest-api#configure-data-retention).
- Updated Chapter 10's "Delta table writes and housekeeping" bullet to mention that the retention window is configurable, and pointed it at Chapter 4, Section 4.1 for the full detail instead of duplicating it.

## 8 August 2026 (Spelling, grammar, and factual review of Chapters 1-10)

### Editorial changes

- Full spelling and grammar pass across Chapters 1-10, mostly correcting artefacts left by direct file edits between sessions. Fixed contraction errors ("its" for "it's") in Chapters 1, 2, and 3; a subject-verb agreement error ("Microsoft do not support" and "the problems just gets worse"); missing articles ("a Mirroring solution", "an F4 capacity"); typos ("replies on" for "relies on", "unnessaray", "over looked", "in in UTC", "executes again Snowflake"); a doubled/garbled sentence in Chapter 4's mirrored-database-item description; and inconsistent product-name capitalisation ("Onelake", "mySQL", "Sharepoint", "Vnet").
- Chapter 3 needed the heaviest cleanup: rewrote a garbled sentence in the Push-Based Mirroring section, fixed a heading mistakenly set to H1 instead of H3 ("Backoff and Polling Cadence"), removed several stray escaped-asterisk artefacts (`\*\*\*`, `\*\*\*\*\*\*`) left over from a broken footnote reference, cleaned up two raw Microsoft Learn page titles used as link text, and removed a trailing stray backslash after the chapter's final navigation line.
- Restored the substance of two footnotes in Chapter 3 that had been reduced to bare, dangling asterisk markers with an orphaned explanation block after the chapter's navigation footer: folded "backoff is significantly more important than normal" and "shortcuts have no mirroring latency, but SQL analytics endpoint or MD Sync latency can still apply" directly into the sentences they annotate, then removed the now-empty footnote block.
- Chapter 5: corrected "The Replication Monitor tab" to "The Replication Status tab" for consistency with the tab's actual name, used correctly everywhere else in the chapter and book.
- Chapter 3: removed a promise that backoff scripts would appear "in the SQL Server 2016-2022 chapter" (Chapter 23), since that chapter does not currently contain them; Chapter 3's own backoff coverage already stands on its own.
- Verified Chapter 6's REST API status enumerations (`Initializing/Initialized/Starting/Running/Paused/Stopping/Stopped` for database-level status, `Initialized/Snapshotting/Replicating/Reseeding/Stopped/Failed` for table-level status) directly against the current Microsoft Learn REST API reference; both are accurate and are intentionally a different, more granular set of values than the Fabric portal's simplified Replication Status tab described in Chapter 5.

## 8 August 2026 (Removed redundant Backoff Algorithm section from Chapter 4)

### Editorial changes

- Removed Section 4.6, "The Backoff Algorithm," from Chapter 4, since the backoff mechanism is already fully covered in Chapter 3's "Backoff and Polling Cadence" section. Removed the section's Figure 4.4 diagram and its now-orphaned asset files.
- Preserved the section's one piece of content not duplicated elsewhere: moved the "Backoff is not a separate status value" clarification into Chapter 5's Replication Status Tab discussion, next to the status meanings it clarifies, with a pointer back to Chapter 3 for the backoff mechanism itself.
- Fixed Chapter 4's Section 4.2 cross-reference, which pointed to the now-removed "Section 4.6 and Chapter 5"; it now points to Chapter 5 only.

## 8 August 2026 (Simplified "change feed" terminology)

### Editorial changes

- Replaced "Fabric mirroring change feed" with the shorter "change feed" throughout the book's discussion of push-based mirroring: Chapter 3's Push-Based Mirroring section (bullets and both source tables), Chapter 11 (Azure SQL Database), Chapter 12 (Azure SQL Managed Instance), Chapter 24 (SQL Server 2025), Chapter 31's Open Mirroring use-case example, and the appendix's replication mechanism table and glossary entry (now "Change feed"). Left the distinct Azure Cosmos DB "Change Feed" reference in the appendix glossary untouched, since that is a different, unrelated Cosmos DB feature that Fabric mirroring explicitly does not use.
- Corrected a malformed Chapter 3 bullet for SQL Server 2025 (introduced by a direct file edit between sessions) that was missing the word "requires," the data gateway detail, and a closing period. It now reads: "SQL Server 2025 uses the change feed and requires Azure Arc plus a data gateway."
- Updated the two affected diagrams (Figure 11.1 and Figure 24.1) to use the shortened "Change feed" label, regenerated through the Excalidraw render pipeline and re-validated as native/editable.

## 8 August 2026 (New Chapter 15: Azure Monitor and Chapter 27: AWS Glue Catalog Mirroring)

### Editorial changes

- Added a new Chapter 15, "Azure Monitor," covering the Mirror Azure Monitor Public Preview feature: its connection-based architecture (no replication pipeline, no data copied), the three access paths (Eventhouse endpoint, OneLake shortcuts from Eventhouse, and Lakehouse shortcuts), authentication modes (workspace identity, service principal, organizational account), and Public Preview constraints including the approximately 500-table limit, new-data-only visibility, read-only access, and the two-step data purge process. Added a new native Excalidraw diagram (Figure 15.1). Sources: [Mirror Azure Monitor data in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/azure-monitor), [tutorial](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/azure-monitor-tutorial), and [troubleshooting guide](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/azure-monitor-troubleshoot).
- Added a new Chapter 27, "AWS Glue Catalog Mirroring," covering the AWS Glue Data Catalog Public Preview connector: its Iceberg REST Catalog architecture, IAM access-key authentication and required permissions, the public-internet-only network requirement, and limitations including the Iceberg-only table format requirement and the 500-table limit. Added a new native Excalidraw diagram (Figure 27.1). Sources: [AWS Glue catalog mirroring overview](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/aws-glue), [tutorial](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/aws-glue-tutorial), and [limitations](https://learn.microsoft.com/en-us/fabric/mirroring/catalog-mirroring/aws-glue-limitations).
- Updated Chapter 2's "Metadata Mirroring" section and comparison matrix to include Azure Monitor and AWS Glue alongside Azure Databricks and Dremio, noting that Azure Monitor uses a different, connection-based mechanism (Log Analytics Delta Parquet storage plus Eventhouse) rather than an Iceberg REST Catalog like Dremio and AWS Glue.
- Updated Chapter 3's shortcut-based source tables and the complete source-to-method reference table to include Azure Monitor and AWS Glue as shortcut-based (metadata mirroring) sources.
- To make room for the two new chapters, renumbered every chapter from the former Chapter 15 (Google BigQuery) through the former Chapter 25 (Dremio Catalog Mirroring) up by one, and every chapter from the former Chapter 26 (open mirroring introduction) through the former Chapter 31 (Common Issues and Troubleshooting) up by two. The book now runs Chapter 1 through Chapter 33: Part 2 (Source-Specific Mirroring Guides) now spans Chapters 11-27, and Part 3 (Open Mirroring) now spans Chapters 28-33. Renamed every affected chapter file and diagram asset folder to match, and updated every cross-reference, figure number, and navigation link throughout the book, including `docs/index.md`, `docs/appendix.md`, and the bare chapter-number tables in Chapter 3's method-comparison tables. Entries in this history log dated before today were not rewritten, since they are a record of the chapter numbers that were correct at the time each entry was written.
- Full validation after the renumbering: 0 broken links, 0 em dashes, chapter sequence 1-33 has no gaps or duplicates, every diagram folder and figure number matches its chapter, and all 53 diagrams (51 existing plus the 2 new) validate as native/editable Excalidraw files.

## 4 August 2026 (Part 2 documentation links and Network and Connectivity sections)

### Editorial changes

- Added Microsoft Learn citations throughout Chapters 11-25 (every source-specific chapter in Part 2). Chapters 14, 16, 17, 18, 21, and 24 had no Microsoft Learn links before this pass; the rest gained additional overview, tutorial, security, and FAQ references alongside their existing limitations links.
- Added a dedicated "Network and Connectivity" section to every chapter in Part 2, answering three questions consistently for each source: whether a data gateway is required (none, on-premises, VNet, or both), whether private endpoints are supported, and whether outbound-restricted or firewalled networks are supported.

### Technical corrections

- Corrected Chapter 16 (Oracle): the chapter previously stated that Oracle Cloud Infrastructure (OCI) hosted databases could use direct connectivity without a gateway. The official Oracle mirroring prerequisites list an on-premises data gateway as required for every supported environment, including OCI, Oracle Database@Azure, and Oracle Exadata, not only on-premises hosts. Updated the Nuances table, Setup Walkthrough, and new Network and Connectivity section accordingly.
- Added previously undocumented gateway support to Chapter 15 (Google BigQuery) and Chapter 18 (MySQL). Both connectors support an on-premises data gateway (BigQuery requires gateway version 3000.286.6 or later) and a VNet data gateway for network-isolated sources; the chapters had only described public/firewall-allow-list connectivity.
- Added the current Private Link caveat to Chapter 21 (Snowflake): native Private Link connectivity between a Fabric workspace and Snowflake is not yet available. A VNet data gateway or on-premises data gateway is the current path for private connectivity.
- Chapter 11 (Azure SQL Database): noted that the `sys.sp_help_change_feed` system stored procedure can be used to inspect the current change feed configuration.

## 4 August 2026 (Chapter 10 SKU correction, FAQ rewrite, and proofreading pass)

### Technical corrections

- Removed the F4096 and F8192 rows from Chapter 10's Fabric capacity SKU table, per direct correction. The current [Understand Microsoft Fabric licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses) page still lists these two SKUs; this book follows the correction that they are not available capacities, so the table now stops at F2048.

### Editorial changes

- Rewrote Chapter 10's "Frequently Asked Questions" section (formerly Section 10.5). Removed the standalone Q&A format and folded its unique details into the sections they belong to: the trial-capacity caveat moved into Section 10.2's mirrored-storage bullet, and the pause/resume troubleshooting link moved into Section 10.3's storage bullet. The remaining FAQ items duplicated points already made elsewhere in the chapter and were dropped rather than repeated. "Planning Capacity for Mirroring" is now Section 10.5.
- Added a new bullet on on-premises and VNet data gateway hosting costs to Section 10.4, and cleaned up two duplicate, ungrammatical blockquotes about capacity throttling that had been introduced by direct edits to the chapter file; folded the single correct point into Section 10.1's capacity requirement sentence instead.
- Ran a full spelling and grammar pass across every page edited this session. Fixes included: two grammar errors in README.md ("one of Product Manager" and "its supports"), a truncated sentence in Chapter 3 missing "+ data gateway", a run-on sentence and two "its/it's" contraction errors in Chapter 3, an inconsistent table row in Chapter 3, a capitalisation slip ("Onelake"), a mangled sentence and inconsistent product name ("Fabric SQL Warehouse" corrected to "Fabric Data Warehouse") in Chapter 4, restored a "Backoff is not a separate status value" clarification in Chapter 4 that had been dropped by a direct file edit, removed two stray `<br />` tags and a broken heading fragment ("### v") in Chapter 4, and an "its/it's" and missing-article fix in Chapter 5.

## 3 August 2026 (New Chapter 10: Billing and Capacity Management)

### Editorial changes

- Added a new Chapter 10, "Billing and Capacity Management," covering Fabric capacity and Capacity Units, what mirroring includes for free (core replication compute, mirrored storage up to 1 TB per purchased CU, Delta table writes and housekeeping), what is billed separately (storage beyond the free allowance or while paused, querying mirrored data, extended capabilities, initial setup), source-side and network costs that sit outside Fabric billing, a short FAQ, and capacity-planning guidance. Sources: [Cost of mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/overview#cost-of-mirroring), [Billing for extended capabilities in Mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-billing), and [Understand Microsoft Fabric licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses). Cross-references the existing Chapter 9, Section 9.4 extended-capabilities billing table rather than duplicating it. Added a new native Excalidraw diagram (Figure 10.1) illustrating what is free compared with what is billed.
- To make room for the new chapter directly after Chapter 9, renumbered every chapter from the former Chapter 10 (Azure SQL Database) through the former Chapter 30 (Common Issues and Troubleshooting) up by one, so the book now runs Chapter 1 through Chapter 31. Part 2 (Source-Specific Mirroring Guides) now spans Chapters 11-25, and Part 3 (Open Mirroring) now spans Chapters 26-31. Renamed every affected chapter file and diagram asset folder to match, and updated every cross-reference, figure number, and navigation link throughout the book, including `docs/index.md`, `docs/appendix.md`, and the bare chapter-number tables in Chapter 3's method-comparison tables. Entries in this history log dated before today were not rewritten, since they are a record of the chapter numbers that were correct at the time each entry was written.

## 3 August 2026 (Chapter 8 metadata sync note)

### Product status

- Added a note to Chapter 8's "SQL Analytics Endpoint" section explaining that query freshness through the endpoint depends on a separate background **metadata sync** process, not only on mirroring replication speed, and that this can add a short additional delay beyond mirroring lag. Mentioned the newer low-latency metadata sync option in preview and the available manual refresh paths (portal, REST API, T-SQL stored procedure). Source: [SQL analytics endpoint metadata sync](https://learn.microsoft.com/en-us/fabric/data-engineering/sql-analytics-endpoint-metadata-sync).

## 3 August 2026 (Chapter 5 corrections: tab name, schema, metrics)

### Technical corrections

- Corrected Chapter 5 and Chapter 30's references from "Monitor replication" to the actual **Replication Status** tab name on the mirrored database item. Updated Figures 5.1 and 5.2 (`diagram-01.excalidraw`, `diagram-02.excalidraw`) to match.
- Corrected and completed the `MirroredDatabaseTableExecution` schema table in Chapter 5 against the [Mirrored database operation logs reference](https://learn.microsoft.com/en-us/fabric/mirroring/monitor-logs). Added the missing `CustomerTenantId` column and removed the "Columns not used for mirroring" section: every column in the reference table is part of the schema, so singling some out as "not used for mirroring" was misleading. Columns marked "Not applicable" in the official reference are now noted as such inline in the main table instead of in a separate section.
- Removed the "Backoff or retry behaviour" row from Chapter 5's Key Metrics table. It is not a status or field visible in the Replication Status tab or in the Workspace Monitoring logging schema, so it did not belong in a table of observable metrics.

## 3 August 2026 (Chapter 5 Alerting section)

### Technical corrections

- Removed Chapter 5's "Azure Monitor Note" section. Fabric Mirroring has no Azure Monitor support to caveat; Workspace Monitoring and the Fabric REST API are the only documented monitoring paths, so the note no longer applies.

### Product status

- Added a new "Alerting with Fabric Activator" section to Chapter 5, describing how to build a Real-Time Dashboard tile over the Workspace Monitoring `MirroredDatabaseTableExecution` table, attach a Fabric Activator rule to it (for example, on `ErrorType` or `ReplicatorBatchLatency`), and trigger an email or Microsoft Teams notification. Sources: [What is Fabric Activator?](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-introduction) and [Create Activator alerts from a Real-Time Dashboard](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/data-activator/activator-get-data-real-time-dashboard).

### Editorial changes

- Updated Figure 5.1 (Monitoring options overview) and Figure 5.2 (Monitoring decision flow) to reference Fabric Activator instead of a generic "scheduled checks and alerts" box and a stale "REST API polling plus Logic Apps or Power Automate" branch.
- Updated the "Monitoring Best Practices" and "Summary" sections to reference Fabric Activator as the recommended alerting mechanism.

## 3 August 2026 (Chapter 5 monitoring correction)

### Technical corrections

- Corrected Chapter 5's claim that mirrored databases are monitored from a "Monitoring tab" in the Fabric portal. There is no such tab. The correct feature is the **Monitor replication** view inside the mirrored database item itself. Added a note distinguishing this from the Fabric-wide **Monitoring hub** (opened via **Monitor** in the navigation pane), which does not cover mirrored databases as an item type. Sources: [Monitor mirrored database replication](https://learn.microsoft.com/en-us/fabric/mirroring/monitor) and [Monitoring hub](https://learn.microsoft.com/en-us/fabric/admin/monitoring-hub).
- Updated Figures 5.1 and 5.2 (`diagram-01.excalidraw`, `diagram-02.excalidraw`) to relabel "Monitoring tab" as "Monitor replication".
- Chapter 30: updated two references from "monitoring dashboard/UI" to the mirrored database item's "Monitor replication" view for consistency.

### Editorial changes

- Added direct links to the Fabric mirroring REST API reference and the Workspace Monitoring setup section in Chapter 5, per general guidance that citing public Microsoft documentation inline helps confirm accuracy.

## 3 August 2026 (OneLake security addition)

### Product status

- Added a new "OneLake Security for Mirrored Databases (Preview)" subsection to Chapter 4, Section 4.5. OneLake data access roles now support all mirrored item types (table/folder-level roles, Viewer/Read-permission scope, Admin/Member/Contributor bypass, DefaultReader role, shortcut inheritance). Sources: [Manage OneLake security for Mirrored Databases (Preview)](https://community.fabric.microsoft.com/t5/Fabric-Updates-Blog/Manage-OneLake-security-for-Mirrored-Databases-Preview/ba-p/5172384) and [Get started with OneLake security](https://learn.microsoft.com/en-us/fabric/onelake/security/get-started-security).

### Editorial changes

- Updated Figure 4.3 (Section 4.5 security diagram) to add a new "OneLake security roles" box in the access-control-layers chain, between "Item permissions" and "SQL permissions on endpoint".

## 3 August 2026 (Backoff correction)

### Technical corrections

- Corrected "Backoff" being presented as a formal replication status. Per the [Monitor mirrored database replication](https://learn.microsoft.com/en-us/fabric/mirroring/monitor) documentation, the real database-level statuses are Running, Running with warning, Stopping/Stopped, Failed, and Paused. Backoff is retry behaviour that happens during Running or Running with warning, not a distinct status value.
- Chapter 4: fixed the "Replication States" table (Section 4.6) to list the five real statuses instead of including "Backoff" as one. Relabelled the "Replicator Lifecycle" diagram and bullet list (Section 4.2) from "Backoff" to "Retrying (backoff)" and added a note that this lifecycle model is conceptual, not the literal portal/API status list.
- Chapter 5: corrected the monitoring status list, which used "Error" instead of the documented "Failed" and omitted "Paused".
- Chapter 30: rewrote the "Backoff State Handling" troubleshooting section (now "Diagnosing Backoff and Retry Behaviour"), which incorrectly told readers to look for a literal "Backoff" status in the monitoring dashboard. Updated the troubleshooting decision tree diagram's "Backoff State" node to "Backoff / Retrying" for the same reason.
- Chapter 16 and the appendix glossary: reworded remaining "backoff state" references for consistency.

## 3 August 2026 (replication model rename and Cosmos DB correction)

### Technical corrections

- Reclassified Azure Cosmos DB back to push-based in Chapter 3 and Chapter 12. Continuous Backup captures every insert, update, and delete asynchronously on the source side without Fabric polling for changes, which fits this book's push definition. This corrects the 1 August 2026 change below, which had reclassified it as pull or polling-based.

### Editorial changes

- Renamed Chapter 3's "planning model" terminology to "replication model" throughout (the book-defined push/pull-or-polling/shortcuts framework, not an official Microsoft taxonomy). Updated the model description, both comparison tables' concern-column headers, and the chapter summary to match.

## 1 August 2026

### Technical corrections

- Corrected Chapter 3's Snowflake polling-sequence diagram (Figure 3.3a): the change batch now flows directly from Snowflake to the OneLake landing zone, instead of an incorrect round trip through the Fabric replicator. Updated the surrounding text to match.
- Reclassified Azure Cosmos DB in Chapter 3 as pull or polling-based rather than push-based. Fabric reads Continuous Backup on its own schedule; the source does not push changes to Fabric. This matches Chapter 12 and removes an inconsistency between Chapter 3's own tables.
- Corrected two inconsistent method labels in Chapter 3's source-to-method reference table (Azure SQL Managed Instance with the SQL Server 2022 update policy, and SQL Server 2016-2022) to read "Pull or polling-based" instead of "Push" and "Poll".
- Fixed placeholder chapter cross-references in Chapter 3's source tables that showed a raw "<br />" instead of the chapter number.

### Editorial changes

- Added a missing "## Overview" section to Chapter 2, matching the pattern used in every other chapter.
- Fixed a mislabeled "## Mirroring" heading in Chapter 2 (now "## Open Mirroring") and cleaned up grammar in that chapter.
- Removed stray "<br />" spacer tags from Chapter 3.

## 30 July 2026

### Product status

- Dremio catalog mirroring now has a published Microsoft Learn tutorial and limitations page. Added a dedicated Chapter 24 covering setup, the 500-table limit, the public-internet-only requirement, and Iceberg-to-Delta conversion. Still Public Preview.
- SharePoint List mirroring is now a native, first-party Public Preview connector (Microsoft's official classification is database mirroring), configured through **+ New → Mirrored SharePoint Online List (preview)** in the Fabric portal. It is no longer a do-it-yourself Open Mirroring pattern built on the Microsoft Graph API. Chapter 19 was rewritten to match.
- SharePoint List mirroring uses a hybrid mechanism: list row data replicates into Delta tables, while Document Library data is exposed through OneLake shortcuts.
- Confirmed Google BigQuery and Azure Database for MySQL mirroring remain Public Preview; production workloads are not supported for BigQuery during preview.
- Confirmed the SQL Server mirroring documentation remains a single article covering both the SQL Server 2016-2022 and SQL Server 2025 paths, consistent with this book's Chapter 21/22 split.

### Technical corrections

- Renumbered the Open Mirroring chapters from 24-29 to 25-30 to make room for the new Dremio chapter (Chapter 24), which now sits at the end of Part 2 alongside the other source-specific guides.
- Removed the Microsoft Graph API delta-query code sample from the SharePoint chapter because the native connector does not require custom extraction code.
- Corrected the Chapter 2 mirroring-type comparison so SharePoint List is listed under database mirroring (with a note on its hybrid shortcut behaviour) rather than as an Open Mirroring example.
- Corrected Chapter 3's method classification so SharePoint List is described as a hybrid case (pull-based for list rows, shortcuts for Document Library files) rather than being omitted from the planning tables.

### Editorial changes

- Updated the appendix source matrix, table of contents, and cross-references throughout the book for the new chapter numbering and the SharePoint List correction.

## 15 July 2026

### Product status

- Google BigQuery and Azure Database for MySQL mirroring are Public Preview.
- Dremio catalog mirroring is Public Preview. A source chapter is planned.
- Azure Databricks catalog mirroring is Generally Available.
- Delta change data feed and Mirroring Views are Preview extended capabilities.
- Mirroring Views supports Snowflake only and refreshes on an approximately 12-hour cycle.
- Billing for extended capabilities resumed during the week of 25 May 2026.
- String values up to 16 MB apply to tables created after 18 November 2025. Earlier tables may need recreation.

### Technical corrections

- Standardised the book on the three Fabric mirroring types: database mirroring, metadata mirroring, and open mirroring.
- Separated SQL Server 2016-2022 from SQL Server 2025 because they use different mirroring paths.
- Corrected Azure Cosmos DB replication to use Continuous Backup.
- Corrected Azure SQL Database replication to use external mirroring rather than user-enabled CDC.
- Corrected Google BigQuery replication to use change history through the `CHANGES` table-valued function.
- Corrected SAP replication to use SAP Datasphere replication flow through ADLS Gen2.
- Replaced the obsolete SQL Server 2016-2022 mirroring-agent model with the supported data-gateway and CDC path.
- Added the Azure Arc, managed identity, and data-gateway requirements for SQL Server 2025.
- Clarified that Azure Databricks and Dremio use metadata synchronisation and OneLake shortcuts rather than copying source data into OneLake.
- Clarified that stopping and restarting database mirroring triggers a full reseed.
- Clarified that the SQL analytics endpoint is read-only for mirrored data.
- Added the warning that source security settings do not propagate automatically to Fabric.
- Updated monitoring guidance to use the mirrored database monitoring view, Workspace Monitoring with Eventhouse and KQL, and the Fabric REST API.
- Removed unsupported ARM and Bicep deployment guidance.

### Editorial changes

- Reworked the contents and appendix around source, mirroring type, replication method, and availability.
- Replaced unsupported or ambiguous claims with public documentation references.
- Began a full style refresh based on the concise, reader-focused format used in the repository README.

## Earlier Product Milestones

- **May 2023**: Microsoft Fabric was announced at Build.
- **November 2023**: Mirroring was previewed at Ignite for Azure SQL Database, Azure Cosmos DB, and Snowflake.
- **March 2024**: Azure SQL Database, Azure Cosmos DB, and Snowflake mirroring entered Public Preview.
- **November 2024**: Azure SQL Database and the Mirrored Database REST API reached General Availability. Azure SQL Managed Instance and Open Mirroring entered Preview.
- **2025**: Azure SQL Managed Instance, Azure Cosmos DB, Snowflake, Azure Database for PostgreSQL, Oracle, SAP, SQL Server, and Azure Databricks metadata mirroring reached General Availability.
- **2026**: Dremio catalog mirroring entered Public Preview, and Fabric SQL Database mirroring reached General Availability.

**Contents:** [Table of Contents](index.md) | **Previous:** [Supported Sources, Mirroring Types, and Reference Tables](appendix.md)
