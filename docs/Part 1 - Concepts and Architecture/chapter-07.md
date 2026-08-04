# Chapter 7: Deploying a Mirrored Database Using CI/CD

> **Part 1: Concepts and Architecture**
>
> **Purpose:** Use this chapter to choose and implement a repeatable deployment pattern for mirrored databases across environments.

---

## Overview

Fabric supports four verified deployment patterns for mirrored databases: Git integration, Deployment Pipelines, REST API scripting, and the Fabric Terraform provider.

[![Figure 7.1 - CI/CD pipeline flow for mirrored databases](../assets/diagrams/chapter-07/diagram-01.png)](../assets/diagrams/chapter-07/diagram-01.excalidraw.png)
*Figure 7.1 - CI/CD pipeline flow for mirrored databases*

---

## CI/CD Support

Fabric supports native source control and promotion workflows for mirrored databases.

### Fabric Git Integration

Fabric workspaces can be connected to a **Git repository** in Azure DevOps or GitHub through the Fabric portal. When configured:

- Workspace items, including mirrored databases, are serialised to Git-compatible JSON files.
- Changes made in the portal can be committed to the connected branch.
- Changes pushed to the branch can be synchronised back to the Fabric workspace.

**Mirrored database artefacts in Git** include:

- `<mirrored-database-name>.MirroredDatabase/` directory
- `item.config.json` for item metadata such as name, type, and description
- `item.definition.json` for the source connection reference, table selection, and mirroring configuration

**Important:** Source credentials are **not** stored in Git. The connection reference, typically the Connection ID, is stored in the item definition, but the actual credentials remain in Fabric's connection store. Because connection IDs differ between environments, they must be parameterised during promotion.

### Fabric Deployment Pipelines

Fabric **Deployment Pipelines** support promotion across environments, usually Development to Test to Production, with:

- Promotion of individual items or all items in a workspace
- Deployment rules for environment-specific values
- A controlled path for validation before production release

---

## Infrastructure as Code

For teams that want scripted deployment, the two current programmatic paths are the **Fabric Terraform provider** and direct **REST API** calls. If you need infrastructure-as-code beyond Terraform, the REST API is the current recommended path for mirrored databases.

### Terraform (Fabric Provider)

```hcl
resource "fabric_mirrored_database" "sales_mirror" {
  workspace_id = var.workspace_id
  display_name = "Sales Database Mirror"

  definition = {
    source_connection_id = var.sql_connection_id
    source_database_name = "SalesDB"
    tables = ["dbo.Orders", "dbo.Customers", "dbo.Products"]
  }
}
```

*Note: Verify resource names and attribute schemas against the current Fabric Terraform provider documentation before deployment.*

### REST API Deployment Script

For maximum control, you can create and configure a mirrored database through the Fabric REST API:

```bash
# Create a mirrored database
curl -X POST "https://api.fabric.microsoft.com/v1/workspaces/${WORKSPACE_ID}/mirroredDatabases" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "Sales Database Mirror",
    "definition": {
      "parts": [
        {
          "path": "mirroring.json",
          "payload": "<base64-encoded-config>",
          "payloadType": "InlineBase64"
        }
      ]
    }
  }'
```

This approach is also the most practical option when you need custom environment substitution, release orchestration, or infrastructure-as-code patterns that are not yet covered by Terraform.

---

## Service Principal and Permissions

Automated deployments should authenticate as a **service principal** rather than a user account.

### 1. Create a Service Principal

In the Azure portal under **Microsoft Entra ID**:
1. Register a new application.
2. Create a client secret or upload a certificate.
3. Record the **Application (client) ID** and **Tenant ID**.

### 2. Grant Workspace Access

In the Fabric portal:
1. Open the target workspace settings.
2. Under **Access**, add the service principal as a **Member** or **Admin**.

### 3. Enable Service Principal Access in Fabric Admin Settings

In the Fabric Admin portal:
1. Go to **Tenant settings** and then **Developer settings**.
2. Enable **Service principals can use Fabric APIs**.
3. Optionally restrict access to a specific security group.

### 4. Connection Permissions

The service principal must also have access to the **Fabric Connection** used by the mirrored database:
- Add the service principal as an owner or user of the connection in **Manage connections and gateways**.

---

## Promotion Between Environments

Promoting a mirrored database between environments mainly involves substituting environment-specific values, especially connection references and source database names.

### Parameterisation Strategy

Use deployment parameters to supply environment-specific values during promotion:

| Parameter | Dev Value | Test Value | Prod Value |
|---|---|---|---|
| `source_connection_id` | `conn-dev-sql-01` | `conn-test-sql-01` | `conn-prod-sql-01` |
| `source_database_name` | `SalesDB_Dev` | `SalesDB_Test` | `SalesDB` |
| `workspace_id` | `ws-dev-xxxxx` | `ws-test-xxxxx` | `ws-prod-xxxxx` |

In Deployment Pipelines, configure these values through **Deployment rules**.

In scripted releases, use environment variables, parameter files, or your CI/CD platform's secret and variable store.

### Promotion Workflow

A typical workflow looks like this:

[![Figure 7.2 - End-to-end CI/CD promotion workflow](../assets/diagrams/chapter-07/diagram-02.png)](../assets/diagrams/chapter-07/diagram-02.excalidraw.png)
*Figure 7.2 - End-to-end CI/CD promotion workflow*

### Post-Deployment Verification

After promotion to a new environment:

1. Verify that the connection resolves to the correct server, database, and stored credentials.
2. Start mirroring if the release process does not already do so.
3. Confirm that the initial snapshot or resumed replication completes successfully.
4. Compare row counts or sample records against the source.
5. Enable or confirm monitoring and alerting. See Chapter 5 for monitoring guidance.

---

## Summary

For mirrored databases, the currently verified deployment patterns are Git integration, Deployment Pipelines, REST API scripting, and the Fabric Terraform provider. The main operational task is parameterising environment-specific values, especially connection IDs, because credentials stay in Fabric rather than in source control.

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 6: Using the Fabric REST API](chapter-06.md) | **Next:** [Chapter 8: Using a Mirrored Database](chapter-08.md)
