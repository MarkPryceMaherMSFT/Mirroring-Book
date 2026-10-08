# Chapter 41: PostgreSQL Polling with impulse_sync

> **Part 3: Open Mirroring**
>
> **Purpose:** Evaluate a lightweight query-based publisher and understand which changes its extraction modes can and cannot observe.

**Part index:** [Chapters in Part 3](readme.md)

---

## Project and Scope

[srutz/impulse_sync](https://github.com/srutz/impulse_sync) is a Node/TypeScript application with an `impulse-sync` CLI and an [MIT licence](https://github.com/srutz/impulse_sync/blob/b5921324d7367c5d2c21c62dfefdce1ba112b417/LICENSE). The inspected revision is `b59213`, with package version 1.0.0.

```text
PostgreSQL query
    -> streamed rows
    -> local Parquet
    -> OneLake upload
    -> local progress marker
```

This is query-based extraction, not PostgreSQL WAL decoding. It can be simpler to host than the Kafka pipeline, but that simplicity changes the completeness guarantees. It is suitable for studying append-only or state-polling patterns, not for assuming every source transaction is captured.

## Choose the Extraction Mode First

| Mode | What it reads | What it does not solve |
|---|---|---|
| Full | Current query results on each run | Removed source rows do not automatically disappear from Fabric |
| Timestamp | Rows beyond a saved timestamp | Timestamp ties, backdated values, late commits and hard deletes need additional design |
| Increasing primary key | Rows beyond the saved key | Updates/deletes to older keys and allocation-versus-commit ordering |

The [run implementation](https://github.com/srutz/impulse_sync/blob/b5921324d7367c5d2c21c62dfefdce1ba112b417/src/sync/run.ts) emits upserts. Full mode is therefore a full reread, not destination snapshot replacement. A destination can retain a row that no longer exists at the source.

## Setup

Follow the repository's installation/build instructions and inspect [package.json](https://github.com/srutz/impulse_sync/blob/b5921324d7367c5d2c21c62dfefdce1ba112b417/package.json) for the supported commands. Configure the source query, tables, key/watermark fields and Fabric credentials. Use a new test destination and the shared setup from [Chapter 29](chapter-29.md).

Choose stable non-null business keys and a query with deterministic types. The implementation infers schema from the first row, so an unrepresentative null, decimal or timestamp in that row deserves testing. Do not infer arbitrary composite-key support from the fact that Fabric supports it.

Keep local output and marker storage durable across process/container replacement. These are part of the implementation's progress model, not disposable temporary files.

## Progress and File Handling

The project saves the maximum extracted source marker after upload. That is a better boundary than acknowledging before attempting delivery, but does not by itself solve ambiguous publication.

The [OneLake upload code](https://github.com/srutz/impulse_sync/blob/b5921324d7367c5d2c21c62dfefdce1ba112b417/src/azure/fabric.ts) creates, appends and flushes the final path. Retrying that operation is not the immutable atomic-publication pattern described in [Chapter 32](chapter-32.md).

File numbering is derived from local Parquet output. Local [sync marker handling](https://github.com/srutz/impulse_sync/blob/b5921324d7367c5d2c21c62dfefdce1ba112b417/src/sync/syncmarkers.ts) can default missing or unreadable state to empty. An operational adaptation should surface lost/corrupt progress rather than silently treating it as a fresh extraction.

Avoid using `singleFileMode` as a repeated-update mechanism for an already processed landing-zone path: it reuses a filename, while Fabric does not interpret overwriting a processed file as a new logical batch.

## Exercises That Reveal the Difference from CDC

Load three rows, update the oldest key, delete another row, then insert two rows sharing a timestamp. Compare each mode against the source. Next, allow a transaction with an earlier allocated key or timestamp to commit after the watermark has advanced.

These exercises explain why adding retries does not make incomplete extraction lossless. For state polling, add an explicit overlap/reconciliation strategy and deletion detection. If every committed change is required, evaluate a source-supported change stream instead.

For publication, adopt durable batch assignments, temporary files, atomic rename and explicit error handling. Monitor extraction progress and Fabric table freshness separately. The project is useful precisely because its compact implementation makes those boundaries visible.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 40: PostgreSQL Through Debezium and Kafka](chapter-40.md) | **Next:** [Chapter 42: Synapse Dedicated SQL Pool Open Mirroring](chapter-42.md)
