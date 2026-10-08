# Chapter 31: Setting Up Open Mirroring: Step-by-Step Configuration

> **Part 3: Open Mirroring**
>
> **Purpose:** Use this chapter to create an Open Mirrored Database, authenticate to its landing zone, and implement a Python extraction and delivery pipeline.

**Part index:** [Chapters in Part 3](readme.md)

---

## Overview

This chapter walks through creating an Open Mirrored Database and implementing a Python-based extraction pipeline.

[![Figure 31.1: Six-step Open Mirroring setup process](../assets/diagrams/chapter-31/diagram-01.png)](../assets/diagrams/chapter-31/diagram-01.excalidraw.png)
*Figure 31.1: Six-step Open Mirroring setup process*

---

## Step 1: Creating an Open Mirrored Database

Use a workspace assigned to an active Fabric capacity. Pausing or deleting the capacity interrupts replication.

1. In the Fabric portal, navigate to your workspace and open the **Create** hub.
2. Select the **Mirrored Database** card for Open Mirroring.
3. Provide a display name (e.g., `MyOpenMirror`).
4. Click **Create**.

Fabric creates the mirrored database item and provisions:
- A **landing zone** path in OneLake
- A **SQL analytics endpoint** (initially empty, until tables are created)

### Retrieving the Landing Zone URL

From the mirrored database item page:
1. Open the **Home** page and locate its details section.
2. Copy the **Landing Zone URL**. This is the OneLake ADLS Gen2-compatible path where your application will write files.

The URL format is:

```text
https://onelake.dfs.fabric.microsoft.com/<workspace-id>/<mirrored-database-id>/Files/LandingZone/
```

---

## Step 2: Authentication

Your application needs a Microsoft Entra identity and permission to write to the landing zone. This example uses a **service principal**:

1. Register an application in Microsoft Entra ID.
2. Configure its credentials. This example reads a client secret from environment variables; do not embed it in source code.
3. Grant **Read and write** on the mirrored database item, or a workspace role such as **Contributor** where broader access is required.
4. For Fabric REST API automation, confirm that the principal is allowed by the **Service principals can call Fabric public APIs** tenant setting.
5. Use a Storage-audience token for OneLake:

```python
import os
from azure.identity import ClientSecretCredential

credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"]
)

# Get a token for OneLake / ADLS Gen2
token = credential.get_token("https://storage.azure.com/.default")
```

Pass `credential`, not the token string, to the storage SDK so it can refresh tokens. Fabric control-plane calls use a separate token for `https://api.fabric.microsoft.com/.default`. See [OneLake authentication](https://learn.microsoft.com/en-us/fabric/onelake/onelake-access-api), [item permissions](https://learn.microsoft.com/en-us/fabric/mirroring/share-and-manage-permissions), and [service principal tenant settings](https://learn.microsoft.com/en-us/fabric/admin/service-admin-portal-developer).

---

## Step 3: Implementing Change Data Extraction

Your application is responsible for extracting changes from the source system. The general pattern is:

1. **First run**: Extract all existing data (full snapshot).
2. **Subsequent runs**: Extract changes after the last durably published source position.

Establish a consistent boundary between snapshot and change capture. Start retaining source changes before or at that boundary, then publish the snapshot followed by changes after it. Independently running a full-table query and then starting CDC can miss changes in between.

### Watermark Strategy

A watermark is a marker that identifies the last processed position in the source change stream. Common watermark types:

| Watermark Type | Example | Notes |
|---|---|---|
| **Timestamp column** | `updated_at >= '2024-11-01 10:00:00'` | Needs a stable tie-breaker, late-commit handling, and separate delete capture; not inherently lossless CDC |
| **Auto-increment ID** | `id > 12345` | Append-only sources only; allocation order is not necessarily commit order, so handle late commits |
| **Log sequence number** | LSN from SQL Server CDC | Precise; requires CDC access |
| **API delta token** | Microsoft Graph delta token | Distinguish intermediate page links from the completed change-tracking token |

Store watermarks in durable storage (e.g., Azure Blob Storage, Azure SQL, or a Fabric Lakehouse table) so they survive application restarts.

Before uploading a batch, durably record its source range, immutable payload, and assigned file sequence. After a successful rename, mark that assignment published and advance the source watermark. A crash between these steps must retry the same assignment, not allocate another file for the same changes. This is publisher implementation guidance, not a Fabric-managed source checkpoint.

---

## Step 4: Uploading Data to the Landing Zone

Write extracted data to the OneLake landing zone using the Azure Data Lake Storage Gen2 (ADLS) SDK.

### Landing Zone Directory Structure

```
Files/LandingZone/
    <table-name>/
        _metadata.json
        00000000000000000001.parquet
        00000000000000000002.parquet
```

### File Format Requirements

- Use **Parquet** or a supported delimited-text format.
- Use contiguous 20-digit, zero-padded file names in the default sequential mode, beginning at 1.
- Put `_metadata.json` inside each table folder.
- This pattern does not use watermark directories or commit marker files.
- For incremental changes, use `__rowMarker__` as the final column: `0` insert, `1` update, `2` delete, or `4` upsert. Use an integer Arrow type in these examples.

Create the table folder and its `_metadata.json` before publishing data. For example, `Files/LandingZone/customers/_metadata.json` contains:

```json
{
  "keyColumns": ["id"]
}
```

For Parquet, column types come from the file, not this metadata. Initial snapshot files normally omit the row marker; later update/upsert rows contain the full row. Chapter 34 covers deletes, schema changes, and alternative detection.

### Python Code Sample

This helper publishes a small, already-serialised Parquet batch to an existing table folder. It assumes one coordinated publisher, default sequential detection, and at most one outstanding assignment per table. Create the folder and metadata first, and persist the exact payload bytes with the assignment before calling it.

```python
import io
from urllib.parse import quote

import pyarrow.parquet as pq
import requests
from azure.storage.filedatalake import DataLakeServiceClient
from azure.core.exceptions import ResourceNotFoundError

def upload_to_landing_zone(
    workspace_id: str,
    database_id: str,
    table_name: str,
    payload: bytes,
    sequence: int,
    credential
):
    """Publish or verify one durably assigned, immutable Parquet batch."""
    if not isinstance(sequence, int) or isinstance(sequence, bool):
        raise ValueError("Sequence must be an integer")
    if not 1 <= sequence < 10**20:
        raise ValueError("Sequence must be positive and at most 20 digits")
    if (not table_name or table_name.startswith("_")
            or table_name in (".", "..")
            or "/" in table_name or "\\" in table_name):
        raise ValueError("Use one table name in the default schema")

    # Reading the complete file detects invalid Parquet before publication.
    data = pq.read_table(io.BytesIO(payload))
    endpoint = "https://onelake.dfs.fabric.microsoft.com"
    service_client = DataLakeServiceClient(
        account_url=endpoint,
        credential=credential
    )

    fs_client = service_client.get_file_system_client(workspace_id)
    base_path = f"{database_id}/Files/LandingZone/{table_name}"
    dir_client = fs_client.get_directory_client(base_path)
    dir_client.get_file_client("_metadata.json").get_file_properties()

    final_name = f"{sequence:020d}.parquet"
    final_client = dir_client.get_file_client(final_name)
    try:
        existing = final_client.download_file().readall()
    except ResourceNotFoundError:
        existing = None
    if existing is not None:
        if existing != payload:
            raise ValueError("Final path contains a different batch; reconcile it")
        return final_name

    temp_name = f"_{final_name}"
    temp_client = dir_client.get_file_client(temp_name)
    temp_client.upload_data(payload, overwrite=True)

    # The condition prevents the REST rename from overwriting a final path.
    relative_path = f"/{workspace_id}/{base_path}"
    response = requests.put(
        endpoint + quote(f"{relative_path}/{final_name}", safe="/"),
        headers={
            "Authorization": "Bearer " + credential.get_token(
                "https://storage.azure.com/.default"
            ).token,
            "x-ms-version": "2020-06-12",
            "x-ms-rename-source": quote(
                f"{relative_path}/{temp_name}", safe="/"
            ),
            "If-None-Match": "*",
        },
        timeout=60,
    )
    response.raise_for_status()

    print(f"Uploaded {data.num_rows} rows as {final_name}")
    return final_name
```

The [ADLS Gen2 rename API](https://learn.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/create?view=rest-storageservices-datalakestoragegen2-2019-12-12) overwrites destinations by default, so the conditional header is important. On a timeout or conflict, reconcile the same durable assignment before retrying. Do not advance the watermark after a failed or ambiguous upload.

The example deliberately reads the existing file to verify an exact retry. For large batches, use a tested durable content-verification design. Validate schema, keys, and row markers before serialisation; this helper is not a complete CDC scheduler or recovery system.

Fabric retains the latest processed sequential data file as a publisher reference. Reconcile that reference with durable assignments; taking the largest visible name and adding one is not enough after a crash. Never overwrite a published path or reset the sequence merely because a listing is empty.

---

## Step 5: Starting and Monitoring Processing

The [portal tutorial](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-tutorial) describes portal-created items as ready to process uploads. For an API-created item, or one not yet started, prepare the landing-zone structure and initial files before starting. Check status rather than assuming that creating the item started replication.

Use these Fabric REST calls with a Fabric-audience token:

```http
POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/mirroredDatabases/{mirroredDatabaseId}/getMirroringStatus
POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/mirroredDatabases/{mirroredDatabaseId}/startMirroring
POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/mirroredDatabases/{mirroredDatabaseId}/getTablesMirroringStatus
```

Wait until initial provisioning leaves `Initializing`; `Initialized` means ready to start. A successful start returns HTTP 200, not proof that every table has finished loading. Poll database and table status, follow table-status pagination, and honour `Retry-After` for HTTP 429. Do not routinely stop and restart an existing Open Mirrored Database: [restart recovery guidance](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices#plan-for-recovery) says that restarts it from the beginning.

Once replication is running, Fabric's background engine:

1. Detects new files in the landing zone directory.
2. Reads the Parquet schema or the delimited-text schema in `_metadata.json`.
3. Applies inserts, updates, deletes, or upserts according to `__rowMarker__`.
4. Updates the Delta log and makes the new data queryable.
5. Moves processed files to an internal cleanup folder and deletes them after the retention period, while keeping the latest processed data file.

Monitor actual processing latency from the mirrored database item or Workspace Monitoring.

---

## Step 6: Verifying the Setup

After the first upload:

1. Return to the mirrored database item page in the Fabric portal.
2. Open **Monitor replication** or **Replication status** in the item. Inspect each table's status and errors, not only the database status.
3. Open the **SQL analytics endpoint** and query the table:

```sql
SELECT TOP 10 * FROM [MyOpenMirror].[dbo].[customers];
```

4. Compare destination row counts and representative values against the source at a consistent checkpoint, allowing for replication lag. The monitoring view's **Rows replicated** counts applied operations, including updates and deletes; it is not the current number of rows in the table.

Also test a full-row update, an upsert of a new key, a key-only delete, and retrying the same publication assignment. If data is present in OneLake but not in SQL, investigate [SQL endpoint metadata synchronisation](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting#data-doesnt-appear-to-be-replicating).

**API references:** [Start mirroring](https://learn.microsoft.com/en-us/rest/api/fabric/mirroreddatabase/mirroring/start-mirroring), [database status](https://learn.microsoft.com/en-us/rest/api/fabric/mirroreddatabase/mirroring/get-mirroring-status), and [table status](https://learn.microsoft.com/en-us/rest/api/fabric/mirroreddatabase/mirroring/get-tables-mirroring-status).

---

## Summary

Open Mirroring setup requires a Fabric item, authentication, source change extraction, per-table metadata, and sequential file delivery. Use temporary names and atomic rename so Fabric never reads a partial file.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 30: What is Open Mirroring and Why It's Useful](chapter-30.md) | **Next:** [Chapter 32: Code Samples and the Fabric Toolbox](chapter-32.md)
