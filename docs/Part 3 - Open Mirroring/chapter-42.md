# Chapter 42: PostgreSQL Through Debezium and Kafka

> **Part 3: Open Mirroring**
>
> **Purpose:** Follow a real PostgreSQL CDC pipeline and understand why source offsets must be tied to successful publication.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project and Data Flow

[DaSenf1860/mirror_postgres](https://github.com/DaSenf1860/mirror_postgres) supplies a Docker-based demonstration and Python publisher under the [MIT licence](https://github.com/DaSenf1860/mirror_postgres/blob/b8ffc51513993386a17c3d3b5af0afe038ff4747/LICENSE). This chapter examines revision `b8ffc51`.

```text
PostgreSQL logical replication
    -> Debezium pgoutput
    -> Kafka topics
    -> Python transformation
    -> Parquet in OneLake
```

Unlike the polling project in [Chapter 43](chapter-43.md), this design obtains change events from logical replication. It can represent deletes, not just rows still present when a query runs. It is nevertheless a demonstration with important startup and acknowledgement behaviour to change before adopting it as a continuous service.

## Setting Up a Controlled Lab

Use the repository's Docker/configuration assets to prepare PostgreSQL, Kafka and the Debezium connector. Configure logical replication and the connector's source permissions for the exact deployment. Set the source tables, Fabric workspace/database identifiers and service-principal credentials. Apply the shared destination setup from [Chapter 31](chapter-31.md).

Read [mirrorpostgres.py](https://github.com/DaSenf1860/mirror_postgres/blob/b8ffc51513993386a17c3d3b5af0afe038ff4747/mirrorpostgres.py) before starting it. Startup creates timestamp-based consumer-group identifiers, recreates the connector and resets target folders. That is a reseeding workflow, not a transparent resume. Use an expendable mirrored database.

The connector configuration uses `snapshot.mode=always`. Source snapshot behaviour, Kafka history and destination reset must be considered together. A fresh consumer group is not a durable continuation of the previous consumer's checkpoint.

## Row Conversion and Keys

The [transformation module](https://github.com/DaSenf1860/mirror_postgres/blob/b8ffc51513993386a17c3d3b5af0afe038ff4747/mirroring_postgres_utils.py) maps snapshot/create/update operations to marker `4` and deletes to `2`. It derives Arrow types from PostgreSQL metadata and appends the row marker last.

The inspected key assumption is hardcoded to `id`. Extend key metadata and event handling together before using a composite key or a differently named primary key. Test a source key change explicitly.

The batch reduction uses maximum `ts_ms` per key. Millisecond timestamps are not unique event sequence numbers; equal timestamps can select the wrong representative event. Preserve the ordering supplied by the source/Kafka partition rather than assuming timestamp reduction always yields the final state.

## Offset Acknowledgement and Publication

In [list_kafka_messages.py](https://github.com/DaSenf1860/mirror_postgres/blob/b8ffc51513993386a17c3d3b5af0afe038ff4747/list_kafka_messages.py), auto-commit is enabled. The consumer returns messages before the caller publishes them. Kafka progress can therefore be independent of OneLake publication.

The [OneLake writer](https://github.com/DaSenf1860/mirror_postgres/blob/b8ffc51513993386a17c3d3b5af0afe038ff4747/mirroring_utils.py) writes directly to final paths with overwrite enabled. It does not implement the complete temporary-upload/atomic-rename/immutable-path protocol from [Chapter 34](chapter-34.md).

For a reliable adaptation, retain stable consumer identity, disable premature acknowledgement, persist each batch's source range and publication assignment, and commit only a contiguous durably published prefix. Preserve the same assignment after an ambiguous upload. Exactly-once delivery is not supplied merely by using Kafka and upsert markers.

## A Practical Hardening Exercise

Create one keyed table and exercise initial rows, an update, a delete and several rapid changes to the same key. Then make OneLake unavailable after Kafka consumption, restart the publisher, and observe source offsets and destination files.

Replace destructive startup with a deliberate initialization/resume distinction. Add failure propagation, typed-schema checks, partition ownership and source-retention monitoring. Do not broaden the advertised source list simply because Debezium supports other databases: their envelopes, types and snapshot semantics must be tested through this writer.

Budget PostgreSQL source overhead, retained WAL, Kafka/Debezium hosting, publisher compute and Fabric consumption. The value of this repository is an inspectable end-to-end CDC architecture, not a promise that every operational responsibility has been solved.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 41: MongoDB Through Change Streams](chapter-41.md) | **Next:** [Chapter 43: PostgreSQL Polling with impulse_sync](chapter-43.md)
