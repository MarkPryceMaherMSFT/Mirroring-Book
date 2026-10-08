# Chapter 44: Choosing a Solution, Shared Lessons, and Further Reading

> **Part 3: Open Mirroring**
>
> **Purpose:** Choose by source requirements and ownership, compare real implementations, and keep articles without located source code separate.

**Part index:** [Chapters in Part 3](readme.md)

---

## What This Comparison Means

This is a source-backed selection guide reviewed on 8 October 2026, not a certification list. A public repository makes implementation inspection possible; its licence defines reuse rights. Neither guarantees support, maintained dependencies or correct recovery under your workload.

The project chapters pin inspected revisions where available and distinguish source-review observations from executed deployments. Blog-only references appear separately below. An article's presence in that table means matching public producer code was not found in this search, not proof that none exists.

## Source-to-Project Comparison

| Source or need | Public implementation | Capture or role | Main qualification | Chapter |
|---|---|---|---|---|
| Build your own publisher | [Microsoft SDK](https://github.com/microsoft/fabric-toolbox/tree/main/tools/OpenMirroringPythonSDK) | OneLake table/file helper | MIT; no source CDC or durable delivery journal | [34](chapter-34.md) |
| SQL Server 2008-2022 target stated by sample | [GenericMirroring](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/GenericMirroring) | Change Tracking polling | MIT POC; version coverage is a project claim, not a tested matrix | [35](chapter-35.md) |
| Excel, CSV, Access, SharePoint Lists | [GenericMirroring adapters](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/GenericMirroring/sources) | File/current-state extraction | Stable keys, paging and missing-row deletes need work; EPPlus licensing applies to Excel | [35](chapter-35.md) |
| Excel and SharePoint | [Toolbox notebooks](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring) | Workbook/list snapshots | MIT POCs; not a complete delta/reconciliation protocol | [36](chapter-36.md) |
| MySQL | [MySQL notebook](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/mysql%20Mirroring) | Triggers and change table | MIT POC; acknowledgement ordering needs hardening | [36](chapter-36.md) |
| Snowflake | [Snowflake notebook](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/Snowflake%20Mirroring) | Source streams | MIT POC; stream replacement/publication boundary matters | [36](chapter-36.md) |
| MariaDB | [MariaDBMirroring](https://github.com/microsoft/fabric-toolbox/tree/main/samples/open-mirroring/MariaDBMirroring) | Binlog, MaxScale and Kafka | MIT publisher; MaxScale runtime separately BSL-licensed | [37](chapter-37.md) |
| BigQuery | [FabricBQSync](https://github.com/microsoft/FabricBQSync) | Spark-based configurable extraction | MIT; choose mirrored target, assess source strategy and partial publication | [38](chapter-38.md) |
| MongoDB | [MongoDB_Fabric_Mirroring](https://github.com/mongodb-partners/MongoDB_Fabric_Mirroring) | Initial scan and change streams | Apache-2.0 publisher; handoff, resume and schema policy need evaluation | [39](chapter-39.md) |
| PostgreSQL CDC | [mirror_postgres](https://github.com/DaSenf1860/mirror_postgres) | Debezium and Kafka | MIT lab; destructive startup and offset acknowledgement need changes | [40](chapter-40.md) |
| PostgreSQL polling | [impulse_sync](https://github.com/srutz/impulse_sync) | Full, timestamp or increasing-key queries | MIT; no delete detection or WAL capture | [41](chapter-41.md) |
| Synapse dedicated SQL pool | [fabric-mirroring-synapse](https://github.com/MarkPryceMaherMSFT/fabric-mirroring-synapse) | Snapshot/diff or append-only watermark | Author-confirmed open source/customer use; named licence not stated in inspected tree | [42](chapter-42.md) |
| Interactive workbook ingestion | [Fabric-OpenMirroring-for-files](https://github.com/nikunj11itdhm/Fabric-OpenMirroring-for-files) | Streamlit/Parquet publisher | MIT; positional keys and missing-row reconciliation need attention | [43](chapter-43.md) |
| Synthetic data and sink tests | [Faker](https://github.com/lmoloney/OpenMirroringFaker), [Shape](https://github.com/sqllocks/shape), [benchmark](https://github.com/mdrakiburrahman/fabric-open-mirroring-benchmark) | Test tooling | MIT projects, not operational database connectors | [43](chapter-43.md) |

The licensing caveats are part of the recommendation, not small print. Check publisher code, dependencies/runtime, source-system rights and hosting costs separately. In particular, do not turn the Toolbox MIT licence into a claim that every dependency is free for commercial use.

## How to Choose

Start with the required source behaviour. If updates and hard deletes matter, reject an append-only extractor unless another mechanism supplies them. If only current state is required, a tested snapshot/diff or polling design may fit without capturing every intermediate transaction.

Next decide where the publisher runs and who owns it. A Fabric notebook consumes Spark resources; an application requires a host; a Kafka bridge requires transport and retention operations. Compare with native connectors from Part 2 before accepting additional ownership.

Finally, evaluate the precise version and your failure cases. A well-explained demonstration can be a better starting point than a larger project whose state model you cannot explain, but neither should receive a production label without evidence.

## Shared Engineering Lessons

**Source position, publication position and destination freshness are different.** A SQL watermark, MongoDB token or Kafka offset says where extraction resumes. A landing-zone filename says which batch was published. Fabric status says what has been processed. Keep all three observable.

**A persisted checkpoint can still be wrong.** Saving offsets while rows remain only in memory is not durable delivery. A shared partition checkpoint must cover a contiguous published prefix across all affected tables, not merely one successful upload.

**Upsert is not exactly-once delivery.** It can prevent some duplicate-key inserts, but an old upsert replayed after a newer update can restore stale data. Marker `0` does not deduplicate even when keys exist.

**Filename discovery is not coordination.** Listing the highest file can help determine progress, but it cannot reserve the next number or identify an already-published source batch. Keep one logical writer per ordered stream, or implement durable coordination.

**Full rereads need deletion reconciliation.** A workbook that becomes shorter or a polled table with hard deletes requires explicit removal records or a deliberate reload. Row positions are not durable identities.

**Source code can lag the current contract.** Some projects put the marker first, publish directly to final names or recreate tables without waiting for disappearance. Keep [Chapter 32](chapter-32.md) and the [current best practices](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices) as the shared reference, while documenting differences honestly.

## Acceptance Exercises

These are exercises to perform against the chosen implementation, not results claimed by this book.

| Exercise | Required evidence |
|---|---|
| Write during initial scan | Final state matches a defined snapshot/change boundary |
| Delete and change a key | Old identity disappears and the new identity has correct values |
| Several rapid changes to one key | Final state follows source order, not timestamp ties or upload timing |
| Fail temporary upload | No incomplete final file is exposed |
| Lose the successful rename response | Retry recognises the same assigned batch without overwrite or duplicate publication |
| Crash after publication, before checkpoint | Recovery reconciles the assigned file before advancing source progress |
| Fail the second file of a batch | Earlier publication is retained and later progress does not skip work |
| Lose local state or source history | Explicit reconciliation/reload, not a silent switch to latest |
| Change schema or key metadata | Compatible changes work; breaking changes follow a planned table rebuild |
| Start two publishers | Ownership prevents conflicting paths or out-of-order logical batches |

Also measure extraction cost, publication latency and query visibility separately. Do not infer row counts from monitoring's processed-operation totals.

## Additional Source-Available Projects

These are useful reading, but an explicit applicable licence was not verified in the inspected trees. They are not included in the licensed recommendation set solely because their code is on GitHub.

| Project | Source or role | Qualification |
|---|---|---|
| [Debezium-Fabric-Open-Mirror-Sink](https://github.com/enekazo/Debezium-Fabric-Open-Mirror-Sink) | Oracle/Debezium Server Java sink | No licence found in inspected tree; review buffered-event acknowledgement |
| [fabric-open-mirroring-sample](https://github.com/cmaneu/fabric-open-mirroring-sample) | .NET/SQLite example | README's MIT link did not resolve to a licence file during inspection |
| [session-open-mirroring](https://github.com/metxito/session-open-mirroring) | SQL Server CDC and control-table examples | No applicable licence found in inspected tree |

The Synapse project is not in this list: its author explicitly confirmed customer reuse and requested its inclusion; [Chapter 42](chapter-42.md) records that basis.

An additional [MIT-licensed Event Hubs bridge](https://github.com/lmoloney/debezium-fabric-mirror) is useful as a recovery case study. Its [consumer](https://github.com/lmoloney/debezium-fabric-mirror/blob/main/src/open_mirroring_debezium/consumer.py) drains table buffers and can advance a shared checkpoint when only some publication succeeds. Do not repeat an at-least-once claim without assessing that control flow. Its inspected Oracle-shaped events do not establish compatibility with every Debezium source.

Do not confuse [microsoft/kafka-sink-ms-fabric](https://github.com/microsoft/kafka-sink-ms-fabric) with an Open Mirroring writer: its Eventhouse/Kusto destination is a different ingestion interface.

## Blog and Article References Without Located Producer Code

These links help readers understand the wider ecosystem, including commercial alternatives. They are not advertised here as free/open-source implementations.

| Source | Article | Code-availability note |
|---|---|---|
| SQL Server / Striim | [Open Mirroring with Microsoft Fabric: Mirror Once, Query Anywhere with Striim](https://medium.com/striim/microsoft-fabric-open-mirroring-mirror-once-query-anywhere-striim-29bfc7f2c01e) | Matching producer code not found; indexed title located, but direct Medium body access returned 403 |
| SQL Server | [Mirroring SQL Server Database to Microsoft Fabric](https://www.striim.com/blog/mirroring-sql-server-database-microsoft-fabric/) | Managed-service walkthrough; matching public producer not found |
| Multiple databases and mainframe | [Qlik + Microsoft Fabric Open Mirroring](https://www.qlik.com/blog/qlik-microsoft-fabric-open-mirroring-the-fast-track-to-real-time-data) | Commercial Qlik integration; matching public producer not found |
| SAP ECC / S/4HANA | [Microsoft Fabric & Open Mirroring: Efficient SAP Data Integration Without ODP](https://theobald-software.com/en/blog/microsoft-fabric-open-mirroring) | Xtract Universal integration; matching public producer not found |
| SAP | [Open Mirroring for SAP sources - dab and Simplement](https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/open-mirroring-for-sap-sources---dab-and-simplement/5172935) | Partner announcement; matching public producers not found |
| SAP | [Microsoft Fabric Open Mirroring: Efficient and innovative](https://www.dab-europe.com/en/articles/microsoft-fabric-open-mirroring-opens-up-new-possibilities-for-data-integration/) | Commercial integration explanation; matching public producer not found |

All "not found" statements are bounded to this research. Reclassify an article when an applicable public implementation and licence become available. A repository containing only sample Parquet files is useful lab material, but is not itself source code for a connector.

## Summary

Choose by source semantics, licence, operational ownership and demonstrated recovery, not by connector count. The SDK and sample projects make Open Mirroring accessible; the shared contract remains the standard against which each publisher must be evaluated.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 43: File Publishing and Open Mirroring Test Tools](chapter-43.md) | **Next:** [Appendix: Supported Sources, Mirroring Types, and Reference Tables](../appendix.md)
