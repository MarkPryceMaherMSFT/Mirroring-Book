# Chapter 39: MariaDB Through MaxScale and Kafka

> **Part 3: Open Mirroring**
>
> **Purpose:** Understand a binlog-to-Kafka implementation and distinguish the open-source publisher from its separately licensed runtime.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project, Architecture and Licensing

[MariaDBMirroring](https://github.com/microsoft/fabric-toolbox/tree/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring) provides a container-based sample:

```text
MariaDB binlog -> MaxScale KafkaCDC -> Kafka
    -> Python consumer -> Parquet -> OneLake landing zone
```

The publisher source is [MIT-licensed](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/LICENSE), but the complete supplied stack is **not unrestricted open source**. Its [Compose file](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring/docker-compose.yml) uses `mariadb/maxscale:latest`, not a pinned version or digest.

The inspected MaxScale 24.02 [licence](https://github.com/mariadb-corporation/MaxScale/blob/844ab7ab5033a54c934757cefea05173ce50934e/licenses/LICENSE2402.TXT) is BSL 1.1 before its version-specific conversion. It permits additional production use with fewer than three server instances, and specifies 10 April 2027 as its change date to GPL-2.0-or-later, subject to the licence's conversion terms. Do not apply that date or grant to every MaxScale version. Pin and review the actual runtime selected for deployment.

This chapter includes the sample because its publisher implementation is public and instructive, with that limitation explicit. The sample collection's proof-of-concept disclaimer still applies.

## Preparing the Lab

Review the Compose services, [MaxScale configuration](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring/config/maxscale/maxscale.cnf), and [consumer configuration](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring/consumer/config.example.yaml). Configure source permissions, Kafka topic/partition, table keys, durable state storage and destination credentials.

The supplied consumer configuration uses local-only output, a 120-second poll interval and a 1,000-row per-table upload cap. Those are sample settings, not Fabric service limits. First prove the locally produced Parquet, then deliberately enable OneLake delivery using the permissions in [Chapter 31](chapter-31.md). Ensure publisher state survives container replacement.

Do not put demonstration passwords on a shared network. Pin container images and dependencies, replace example credentials and restrict exposed ports before expanding the lab.

## Source Capture and State

The [consumer](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring/consumer/consumer.py) contains more state machinery than a simple notebook. `snapshot_source_tables` starts a consistent-snapshot transaction and records GTID-related progress. `load_state` and `save_state` manage offsets, file counters, schemas and bootstrap state, using a temporary state file and replacement.

`normalize_row` maps change events, including deletes; `flush_table_rows` writes sequential Parquet files. Snapshot output omits the row marker, while incremental output places it last. The local helper adds operations such as `upload_bytes`; those are extensions, not methods available in the standalone SDK from [Chapter 36](chapter-36.md).

A consistent source transaction plus a subsequently read global GTID does not by itself prove an exact scan/change-stream handoff under concurrent commits. Test the boundary against the actual server topology and binlog fields. The [GTID utilities](https://github.com/microsoft/fabric-toolbox/blob/b0183fb1841367fd0eaa4aae28949f7911ff4f05/samples/open-mirroring/MariaDBMirroring/consumer/gtid_utils.py) also deserve attention for multidomain values.

## Recovery Work to Understand

In the inspected loop, offsets advance while rows are buffered. A flush can publish only the configured prefix while remaining rows stay in memory, after which state is saved. A saved offset must never acknowledge rows that exist only in memory. Crash recovery needs a durable pending batch or a checkpoint restricted to the contiguous published prefix.

File collision handling and counters are also not a complete source-batch journal. An existing destination filename must be reconciled with its assigned payload, not simply bypassed by choosing a new number. See the [publication design](chapter-31.md) for the distinction.

The consumer manually assigns one partition. Do not infer distributed ownership or failover from Kafka's presence. The destination-validation path can select local-only operation, so monitoring must distinguish local exports from actual Fabric delivery.

## What to Measure and Test

Track binlog/GTID progress, Kafka lag, durable pending batches, destination publication and Fabric table freshness separately. Test a crash with more than one flush worth of buffered rows, destination unavailability, retained Kafka history expiry, and several events for one key.

The architecture demonstrates useful separation between capture, transport and publishing. It does not establish exactly-once delivery, full topology support or an unrestricted free runtime. Budget source resources, Kafka/MaxScale hosting, state storage, network transfer and Fabric analytics separately.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 38: Toolbox Notebook Solutions](chapter-38.md) | **Next:** [Chapter 40: BigQuery with FabricBQSync](chapter-40.md)
