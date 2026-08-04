# Microsoft Fabric Mirroring - Complete Guide

This guide covers the main mirroring patterns in Microsoft Fabric, source-specific setup, and open mirroring implementations. Use Part 1 for concepts and architecture, Part 2 for source guides, and Part 3 for open mirroring and partner-led integrations.

Fabric includes 1 TB of free mirrored storage per purchased CU. An F2 capacity includes 2 TB, an F4 capacity includes 4 TB, and the allowance scales with capacity.

![index diagram 1](assets/diagrams/index/diagram-01.png)(Mirroring Overview)

* Database mirroring copies source data into Delta tables in OneLake.
* Metadata mirroring syncs catalog metadata and uses OneLake shortcuts to access the source data.
* Open mirroring accepts developer-defined or partner-defined change data in a landing zone that Fabric processes.

## Table of Contents

> **Note:** Chapters are published one at a time. A chapter title is a link once it has been published; otherwise it is shown as plain text.

### Part 1: Concepts and Architecture

* [Chapter 1: Introduction to Fabric Mirroring](Part%201%20-%20Concepts%20and%20Architecture/chapter-01.md)
* Chapter 2: Types of Mirroring in Fabric
* Chapter 3: Methods of Mirroring: Push, Pull or Polling, and Shortcuts
* Chapter 4: The Anatomy of a Mirrored Database
* Chapter 5: Monitoring a Mirrored Database
* Chapter 6: Using the Fabric REST API
* Chapter 7: Deploying a Mirrored Database Using CI/CD
* Chapter 8: Using a Mirrored Database
* Chapter 9: Extended Capabilities
* Chapter 10: Billing and Capacity Management

### Part 2: Source-Specific Mirroring Guides

* Chapter 11: Azure SQL Database
* Chapter 12: Azure SQL Managed Instance
* Chapter 13: Azure Cosmos DB
* Chapter 14: Azure Databricks (Unity Catalog) - metadata mirroring
* Chapter 15: Google BigQuery - Public Preview
* Chapter 16: Oracle
* Chapter 17: PostgreSQL
* Chapter 18: MySQL - Public Preview
* Chapter 19: SAP
* Chapter 20: SharePoint List - Public Preview
* Chapter 21: Snowflake
* Chapter 22: SQL Server 2016–2022
* Chapter 23: SQL Server 2025
* Chapter 24: Fabric SQL Database
* Chapter 25: Dremio Catalog Mirroring - Public Preview

> **Note:** Chapter 14 and Chapter 25 cover metadata mirroring rather than database mirroring. Chapters 15, 18, 20, and 25 cover Public Preview sources.

### Part 3: Open Mirroring

* Chapter 26: What is Open Mirroring and Why It's Useful
* Chapter 27: Setting Up Open Mirroring: Step-by-Step Configuration
* Chapter 28: Code Samples and the Fabric Toolbox
* Chapter 29: Use Cases and Examples
* Chapter 30: Metadata and Change Files
* Chapter 31: Common Issues and Troubleshooting

### Appendix

* [Appendix: Supported Sources, Mirroring Types, and Reference Tables](appendix.md)
* Book Update History
