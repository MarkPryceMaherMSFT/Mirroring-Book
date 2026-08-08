# Chapter 10: Billing and Capacity Management

> **Part 1: Concepts and Architecture**
>
> **Purpose:** This chapter explains what Fabric charges for when you use mirroring, what stays free, and how to plan capacity so mirroring costs stay predictable.

***

## Overview

Chapter 9 covered the optional, paid extended capabilities layered on top of mirroring. This chapter steps back and covers the full cost picture: what a Fabric capacity is, what mirroring includes for free, what triggers a charge, and where to look when you need to plan or explain a bill.

The short version is that core mirroring itself is designed to be low-cost. Fabric does not charge for the background compute that replicates your data, and it gives you a free storage allowance tied to the size of your capacity. Charges start when you go past that storage allowance, when you query the mirrored data, when you enable extended capabilities, or when your capacity is paused while data still sits in OneLake.

[![Figure 10.1: What mirroring includes for free compared with what is billed separately](../assets/diagrams/chapter-10/diagram-01.png)](../assets/diagrams/chapter-10/diagram-01.excalidraw.png)
*Figure 10.1: What mirroring includes for free compared with what is billed separately*

***

## 10.1 Fabric Capacity and Capacity Units

Every Fabric workload, including mirroring, runs on a **Fabric capacity**. A capacity is a dedicated pool of compute measured in **Capacity Units (CUs)**, purchased as an F SKU.

| SKU   | Capacity Units (CUs) |
| ----- | -------------------: |
| F2    |                    2 |
| F4    |                    4 |
| F8    |                    8 |
| F16   |                   16 |
| F32   |                   32 |
| F64   |                   64 |
| F128  |                  128 |
| F256  |                  256 |
| F512  |                  512 |
| F1024 |                 1024 |
| F2048 |                 2048 |

> A **running** Fabric capacity is required to set up and operate mirroring, and the capacity must not be throttled when you configure a new mirror. If a capacity is paused or deleted, mirroring stops replicating data, even though the background replication compute itself does not consume capacity units while it runs. See [Understand Microsoft Fabric licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses) for the full SKU and licensing reference.

***

## 10.2 What Mirroring Includes for Free

For database mirroring and open mirroring, Fabric compute and OneLake storage are free up to a capacity-based limit:

* **Mirrored storage**: Fabric gives you **1 TB of free mirrored storage for every Capacity Unit you purchase**. An F64 capacity includes 64 TB of free mirrored storage, exclusive to mirroring. This allowance is calculated at the capacity level, not per mirrored database, so it is shared across every mirror on that capacity. The allowance is tied to a purchased capacity SKU, so confirm current trial-capacity behaviour on the [Microsoft Fabric pricing page](https://azure.microsoft.com/en-us/pricing/details/microsoft-fabric/) before relying on it for a trial.
* **Background replication compute**: The Fabric compute that reads source changes and writes them into OneLake does not consume capacity units. This is the core replication engine covered throughout this book.
* **Delta table writes and housekeeping**: Writing Delta table data as mirroring processes changes, and the automatic optimisation and vacuum of those Delta tables, do not consume capacity separately from the free replication process.

See [Cost of mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/overview#cost-of-mirroring) for the current official description of what is included.

***

## 10.3 What Is Billed Separately

Several things fall outside the free allowance:

* **Storage beyond the free limit**: If your mirrored data exceeds the free terabyte-per-CU allowance, the excess is billed as standard OneLake storage, at a pay-as-you-go rate per GB. Storage is also billed while a capacity is **paused**, since the mirrored data still sits in OneLake even though replication is not running. See [Changes to Fabric capacity](https://learn.microsoft.com/en-us/fabric/mirroring/troubleshooting#changes-to-fabric-capacity) for how a pause and resume affects a mirror.
* **Querying mirrored data**: Reading mirrored tables through the SQL analytics endpoint, Power BI, or Spark consumes capacity like any other Fabric query. Only the replication process itself is free. Requests made directly against OneLake for mirrored data are billed as normal OneLake transactions. See [OneLake consumption](https://learn.microsoft.com/en-us/fabric/onelake/onelake-consumption) for the transaction-level rates.
* **Initial setup and preview**: Connecting to a source and previewing its data while configuring a mirror uses a running capacity and consumes Capacity Units for that setup activity, distinct from the ongoing free replication process.
* **Extended capabilities**: Delta change data feed and Mirroring Views are optional, paid features billed through the `DataMovementIncrementalCopy` meter at a rate of 3 CU-hours, with per-second billing granularity. Chapter 9, Section 9.4 covers this billing model in full, including the unified-charge rule when both capabilities are enabled together. See [Billing for extended capabilities in Mirroring](https://learn.microsoft.com/en-us/fabric/mirroring/extended-capabilities-billing) for the official reference.

***

## 10.4 Source-Side and Network Costs

Fabric's billing model only covers what happens inside Fabric. Three categories of cost sit outside it entirely, and are worth planning for separately:

* **Source system load**: The mechanism a source uses to capture changes, such as CDC, logical replication, or a change feed, runs on the source platform and can add CPU, I/O, or storage overhead there. The scale of that overhead depends on the number of tables mirrored and the volume of changes, and varies by source. See the relevant source chapter in Part 2 for source-specific considerations.
* **Egress charges**: Moving data out of a source cloud provider's region or data centre can incur that provider's own egress or data-transfer charges. These are billed by the source platform, not by Fabric.
* **On-premises data gateway costs**: Sources that connect through an on-premises data gateway or VNet data gateway need that gateway infrastructure provisioned and maintained separately. The hosting and compute cost of running the gateway is not part of Fabric billing.

***

## 10.5 Planning Capacity for Mirroring

* **Estimate mirrored data volume** for each source you plan to mirror, and compare the total against the free-storage allowance of your target capacity SKU before committing to a size.
* **Track storage growth over time**, since ongoing inserts, updates, and retained change history can grow mirrored storage well beyond the initial snapshot size.
* **Monitor consumption** using the Fabric Capacity Metrics app, which reports storage and CU usage at the capacity level across every workload sharing that capacity, not mirroring alone.
* **Separate mirroring capacity from heavy query workloads** where practical, since SQL, Power BI, and Spark queries against mirrored data consume the same capacity as everything else running on it.
* **Revisit extended capabilities deliberately**. Because Delta change data feed and Mirroring Views bill separately, enable them only where the downstream use case needs them, rather than by default on every mirror.

***

## Summary

Core mirroring compute is free, and mirrored storage is free up to one terabyte per purchased Capacity Unit. Charges begin when you exceed that storage allowance, pause a capacity while data remains in OneLake, query mirrored data, enable extended capabilities, or account for source-side, network, and gateway costs outside Fabric. Plan capacity by estimating mirrored data volume against your SKU's free allowance and monitoring growth over time.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 9: Extended Capabilities](chapter-09.md) | **Next:** Chapter 11: Azure SQL Database
