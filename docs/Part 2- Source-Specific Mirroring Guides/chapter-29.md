# Chapter 29: SAP Business Data Cloud Connect for Microsoft Fabric

> **Part 2: Source-Specific Mirroring Guides**
>
> **Purpose:** Understand the announced SAP Business Data Cloud Connect integration, distinguish governed data-product sharing from SAP database mirroring, and prepare an evidence-based implementation decision.

**Part index:** [Chapters in Part 2](readme.md)

---

## Overview and Availability

**Evidence reviewed: 8 October 2026.** SAP Business Data Cloud (SAP BDC) Connect for Microsoft Fabric was announced at Microsoft Ignite in November 2025. The announced capability is **bidirectional, zero-copy sharing**: SAP BDC data products become accessible through Microsoft OneLake, and data sets shared from OneLake become available in SAP BDC.

**Do not treat this chapter as a released setup tutorial.** A Fabric-specific public implementation guide, supported-region matrix and GA confirmation were not verified. The original Q3 2026 target has passed, but elapsed time is not evidence of release. A later SAP-hosted answer gives a different planned date:

| Evidence | What it establishes | What it does not establish |
|---|---|---|
| [SAP announcement, 18 November 2025][sap-announcement], also [published by Microsoft][ms-announcement] | Planned bidirectional, zero-copy sharing; original GA target **Q3 2026** | Actual GA, a usable connector, supported regions or configuration steps |
| [SAP Community answer, 31 August 2026][sap-status], by Thierry Audas, identified on the page as a Product and Topic Expert | Revised statement that GA is **planned for the end of Q1 2027**, expressly subject to change | A release notice, contractual commitment or confirmation of public-preview access |
| [Current SAP BDC Connect provisioning documentation][sap-provisioning] and [consumer-access documentation][sap-access] | Public procedures name **Databricks, Snowflake and Google BigQuery** as partner systems | Those procedures do not name Microsoft Fabric; their availability cannot be transferred to Fabric |
| [Microsoft's current SAP integration guidance][ms-sap-options] | Separately identifies Datasphere mirroring as GA and BDC Connect for Fabric as announced, with future-looking availability wording | A Fabric BDC Connect GA announcement; its “later this year” wording does not resolve the conflicting roadmap dates |

The [Fabric release-status page][ms-whats-new] reviewed for this chapter did not provide a BDC Connect launch confirmation. **Working classification: announced integration; latest explicit revised target found is end-Q1 2027, not a verified release.** Recheck both vendors before procurement or deployment. This finding does not rule out an invitation-only programme.

---

## Architecture and Terminology

“SAP BDC mirroring” can be a useful search phrase, but it is **not an established native database-mirroring source name** in the evidence above. This chapter covers a governed sharing pattern, not an instruction to create a new **Mirrored SAP BDC** item.

The announced logical directions are:

```text
SAP source preparation / data-product production
    -> SAP BDC provider-owned data product
    -> announced BDC Connect sharing -> Fabric / OneLake consumer

Fabric / OneLake provider-owned data set
    -> announced BDC Connect sharing -> SAP BDC consumer
```

This is a conceptual architecture, not a verified deployment diagram. It deliberately leaves the connector's transport, identity exchange and Fabric item type unspecified.

- **Provider ownership remains important.** The provider publishes an approved data product or data set; the consumer uses the exposed data. Sharing does not make Fabric the owner of the SAP application's transactional state.
- **Bidirectional means two sharing directions.** It does not mean transactional writeback to SAP, conflict resolution or bidirectional row synchronisation. Each direction needs its own supported publication and consumption contract.
- **Zero-copy describes the sharing boundary.** It does not prove that SAP source extraction, data-product materialisation, caching, network transfer, query execution or downstream derived tables are absent or free.
- **Business semantics need validation.** “Semantically rich” does not promise automatic conversion of every SAP calculation, hierarchy, currency rule or authorisation into a Power BI semantic model.
- **Do not infer the mechanism.** The reviewed Fabric-specific announcements do not establish Delta Sharing configuration, a OneLake shortcut type, a catalog-mirroring item, SQL endpoint creation or a query-federation engine. Other partners' working implementations are not a Fabric specification.

The wider sharing/federation family is the useful architectural comparison: access governed provider data instead of automatically maintaining another database replica. The precise Fabric implementation remains a documentation gap.

### Difference from Chapter 20

| Concern | [Chapter 20: SAP via Datasphere](chapter-20.md) | This chapter: BDC Connect for Fabric |
|---|---|---|
| Integration unit | SAP objects selected in Datasphere Replication Flows | Announced SAP BDC data products and OneLake data sets |
| SAP-to-Fabric path | SAP source → Datasphere → ADLS Gen2 Parquet → Fabric replication engine → OneLake Delta tables | Announced provider-to-consumer zero-copy sharing; Fabric-specific implementation not verified |
| Fabric setup | Lakehouse shortcut to the landing container, then **Mirrored SAP** | No verified Fabric-specific wizard or item type to document |
| Data responsibility | SAP/Datasphere operates extraction; Fabric maintains a replica from landed files | Provider maintains the published data; consumer governs its use and dependent outputs |
| Direction | The documented replication path is into Fabric | Both sharing directions announced; not application writeback |
| Commercial basis | Datasphere Premium Outbound Integration plus applicable storage/query costs | Obtain the BDC Connect and Fabric terms; do not inherit Chapter 20's pricing assumptions |

Microsoft's [SAP mirroring overview][ms-sap] documents Chapter 20's two-stage replication path. Use that chapter when that is the approved requirement; do not rename its ADLS shortcut as BDC Connect.

---

## Provider and Consumer Responsibilities

The following is a **proposed ownership model**, not a list of product roles or permissions.

| Owner | Responsibility to agree before implementation |
|---|---|
| SAP application/data-product owner | Authoritative source, approved product/version, business keys, refresh contract, semantics, sensitive fields and upstream quality |
| SAP BDC administrator and data steward | Relevant formations and product availability, publication/access approval, supported partner connection and withdrawal process |
| Fabric administrator and data owner | Confirmed feature access, tenant/workspace/capacity eligibility, authorised consumers, supported analytical surfaces and dependent items |
| Identity, security and network teams | Actual authentication model, least privilege, trust boundaries, approved network routes and negative-access tests |
| Commercial and compliance owners | Subscription entitlements, downstream-use rights, residency, charging, retention and support ownership |
| Joint operations team | End-to-end freshness, incident routing, schema/version changes, credential lifecycle and revocation evidence |

For **OneLake → SAP BDC**, Fabric becomes the provider and SAP BDC the consumer. Reversing the direction does not automatically reverse or reuse the same permissions.

---

## Prerequisites: Verified General Guidance and Fabric Gaps

SAP's public BDC Connect documentation is useful preparation, but currently documented partner procedures must not be presented as an executable Fabric recipe.

| Area | Verified public guidance | Additional evidence required for Fabric |
|---|---|---|
| Source preparation | [SAP provisioning guidance][sap-provisioning] requires relevant source systems in the formation for SAP-managed products. Custom Datasphere products use Datasphere's embedded object store and do not require that source-system step. | Exact supported products, versions, publication conditions and source-to-product freshness; not every SAP table is automatically a shareable product |
| Regions and cloud placement | SAP requires BDC Connect and the system sharing the product to align with the product's region and hyperscaler. Its general guidance allows vendor systems to be elsewhere. | A **Fabric-specific** supported region/cloud/tenant combination and any cross-region restrictions; general partner allowances do not approve a Fabric topology |
| Service availability | [SAP's data-centre table][sap-regions] lists BDC components and warns that feature availability can differ within a region. | A “Yes” for **SAP BDC Connect** is not confirmation that its Fabric integration is enabled there |
| Access and permissions | [SAP consumer-access guidance][sap-access] requires an access request and approval, then partner-specific consumption instructions. | Exact SAP and Fabric roles, service identity or delegated identity, supported credential type and permissions needed to create/use the connection |
| Network | [SAP's network reference][sap-network] describes region/provider-specific NAT egress and domains; the publishing instructions explicitly name other partners. | Fabric-specific endpoints, connection direction and support for private networking, gateways or restricted outbound access; do not copy another partner's allowlist |
| Licensing and cost | [SAP Connect metering][sap-metering] points to capacity-unit usage and the service description. SAP provisioning says billing follows actual usage, not the assigned quota. | Fabric eligibility/SKU, SAP entitlement, network/query/cache charges and the contract's permitted downstream uses |

**Security is not automatic permission inheritance.** SAP application roles, BDC publication rights and Fabric consumer permissions are different control points. Do not assume that SAP row filters, column restrictions or sensitivity labels become equivalent Fabric policies.

**Commercial review must cover use, not just access.** Microsoft's [SAP integration guidance][ms-sap-options] explicitly directs readers to SAP terms for limitations on downstream data solutions. SAP's general provisioning page permits caching for applicable use cases; that does not establish unlimited redistribution, an unrestricted replication licence or Fabric-specific terms.

Public SAP Help pages were readable for this review. Their supporting SAP Notes, including Note **3699756** referenced for placement details, may require customer access. Have an authorised SAP administrator review those notes; this chapter neither bypasses that boundary nor substitutes guessed requirements.

---

## Readiness Checklist — Not an Executable Setup Procedure

There is no verified Fabric-specific public walkthrough to reproduce. Complete these gates **before** opening a production change:

1. **Confirm the actual release.** Obtain a current official Fabric-specific support document or release notice, the applicable preview/GA terms and confirmation for the intended tenants. Record the date and support route; the November 2025 announcement alone is insufficient.
2. **Choose the direction and use case.** Record whether the requirement is BDC → OneLake, OneLake → BDC or two separately governed shares. Reject “bidirectional” as shorthand for updating SAP business transactions.
3. **Approve the provider asset.** Identify the product/data set, owner, version, schema, keys, classification, business definitions and refresh contract. Have the provider prove the data is available and current before investigating a consumer connection.
4. **Approve the deployment and commercial boundaries.** Obtain the supported region/tenant pairing, entitlements, permitted processing and retention, and an estimate for source preparation, sharing, networking and analytics.
5. **Obtain the real connection specification.** Require vendor documentation for the protocol, supported item type, identities, minimum permissions and network paths. Do not generate a Delta Sharing profile, create a shortcut, register an application or assign broad grants merely because another partner uses that approach.
6. **Prepare a non-production acceptance plan.** Agree a small approved product, authorised and unauthorised test identities, freshness thresholds, expected semantics, revocation criteria and support escalation evidence.
7. **Implement only the released procedure.** Once the documentation and tenant feature agree, follow that procedure and retain the identifiers and configuration decisions. If they do not agree, stop at readiness rather than substituting Chapter 20's wizard.

**Screenshots:** no verified, explicitly reusable Fabric BDC Connect setup screenshot was identified. No SAP News image or unrelated **Mirrored SAP** wizard is reproduced as a substitute.

---

## Validation and Operational Handover

The checks below are **acceptance criteria for a future supported deployment**, not claims that particular monitoring pages or APIs already exist.

| Check | Evidence to retain |
|---|---|
| Product identity and semantics | Provider-approved asset/version; known business keys and values; correct decimals, dates, units, currencies and any promised metadata |
| Initial readability | Successful read using the intended consumer identity and supported analytical surface, not only an administrator's browse session |
| Freshness | Provider publication/refresh boundary and consumer observation time; compare counts at a common boundary rather than across a changing source |
| Changes and lifecycle | Approved changes made through the provider's normal process; observe only update/delete/schema behaviour the released product contract promises |
| Both directions | Separate proof for each supported share; successful SAP-to-Fabric access does not prove Fabric-to-SAP support |
| Authorisation | An authorised consumer succeeds and an unauthorised consumer fails; test any promised row/column restrictions rather than assuming propagation |
| Cost and performance | Observed provider workload, query capacity, network/caching behaviour and agreed charges; zero-copy is not a performance or cost SLA |

Separate **upstream product freshness** from **sharing/consumer visibility**. If SAP BDC's product is already stale, troubleshooting Fabric cannot repair its source preparation. Likewise, a current product does not prove that every dependent model or cached result is current.

At handover, record provider and consumer identifiers, product version, region, approved uses, identities and expiry owners, dependencies, observed freshness, cost owners and support contacts. Keep secrets out of that inventory.

### Revocation, Change and Retirement

- Agree how the provider withdraws access and how consumer dependencies are discovered. In a controlled test, use the released revocation procedure and verify that new reads by the former consumer fail within the documented interval.
- Do not equate removing a consumer item with revoking a provider-side share. Conversely, revoking a share does not establish deletion of previously exported data, caches, derived tables or reports; manage those under the approved retention policy.
- Treat breaking schema changes, product-version retirement and changes of owner, region or tenant as coordinated changes. Test downstream dependencies before retiring the old contract.
- Use documented credential rotation, interruption and recovery behaviour. Do not invent database CDC retention windows, log offsets or “restart mirroring” recovery for a sharing integration.

---

## Troubleshooting and Common Misidentifications

| Symptom or claim | First response |
|---|---|
| “It was due in Q3 2026, so it must be GA.” | Check the later availability evidence and current tenant-specific support; the revised Q1 2027 statement is also a plan |
| No SAP BDC Connect option in Fabric | Verify whether access is actually offered. Do not change tenant security settings or create **Mirrored SAP** to manufacture the missing connector |
| SAP BDC Connect exists in SAP for Me, but Fabric is absent from partner instructions | General BDC Connect provisioning is not evidence of Fabric onboarding support; consult both vendors |
| A Databricks/Snowflake/BigQuery tutorial works | It validates that partner, not Fabric; do not copy connection identifiers, token formats or grants across products |
| A lesson shows SAP Datasphere consuming OneLake | That is a different integration pattern, not proof that BDC Connect for Fabric has launched |
| An approved consumer cannot read a product after supported setup | Separate product availability, access approval, connection identity, consumer permissions and network reachability; capture non-secret error/correlation details |
| Data is readable but old or semantically wrong | Compare against the provider's published product/version and refresh contract before investigating consumer caching or downstream models |
| Access is revoked but a report still displays data | Distinguish a new provider read from retained/imported/derived results; verify revocation and retention separately |

For the existing Datasphere replication-flow/ADLS path, use [Chapter 20](chapter-20.md). SAP's [Fabric integration learning lesson][sap-learning] discusses Datasphere federation and replication; it is not a Fabric BDC Connect setup guide.

---

## References and Recheck Points

Public evidence reviewed **8 October 2026**:

- **Original announcement:** [SAP News, November 2025][sap-announcement] and [Microsoft's joint announcement][ms-announcement].
- **Later planning update, not a release notice:** [SAP Community GA question and 31 August 2026 answer][sap-status].
- **Current SAP product documentation:** [Provisioning and supported partners][sap-provisioning], [access approval][sap-access], [data-centre availability][sap-regions], [network requirements][sap-network] and [metering][sap-metering].
- **Microsoft status/comparison:** [SAP integration options][ms-sap-options], [Fabric release-status page][ms-whats-new] and [SAP Datasphere mirroring overview][ms-sap].

[sap-announcement]: https://news.sap.com/2025/11/sap-bdc-connect-for-microsoft-fabric-business-insights-ai-innovation/
[ms-announcement]: https://community.fabric.microsoft.com/blog/fbc_fabricupdatesblogs/sap-and-microsoft-accelerate-business-insights-and-ai-innovation-with-sap-busine/5172482
[sap-status]: https://community.sap.com/t5/data-and-ai-professionals-q-a/sap-bdc-connect-for-microsoft-fabric-ga/qaq-p/14469254
[sap-provisioning]: https://help.sap.com/docs/business-data-cloud/administering-sap-business-data-cloud/provisioning-sap-bdc-connect
[sap-access]: https://help.sap.com/docs/business-data-cloud/sap-business-data-cloud-connect/obtaining-access-to-data-products-through-sap-bdc-connect
[sap-regions]: https://help.sap.com/docs/business-data-cloud/product-availability-information/data-center-availability-in-sap-business-data-cloud
[sap-network]: https://help.sap.com/docs/business-data-cloud/sap-business-data-cloud-connect/domains-and-ip-ranges-for-sap-business-data-cloud-connect
[sap-metering]: https://help.sap.com/docs/business-data-cloud/sap-business-data-cloud-connect/sap-business-data-cloud-connect-metering
[ms-sap-options]: https://learn.microsoft.com/en-us/azure/data-factory/sap-change-data-capture-introduction-architecture
[ms-whats-new]: https://learn.microsoft.com/en-us/fabric/fundamentals/whats-new
[ms-sap]: https://learn.microsoft.com/en-us/fabric/mirroring/sap
[sap-learning]: https://learning.sap.com/courses/mastering-sap-data-architecture/integrating-sap-business-data-cloud-with-microsoft-fabric_e4385b8a-0ecd-4cb2-a6c7-c4ca866e9885

**Contents:** [Table of Contents](../index.md) | **Previous:** [Chapter 28: Dataverse Link to Microsoft Fabric](chapter-28.md) | **Next:** [Chapter 30: What is Open Mirroring and Why It's Useful](../Part%203%20-%20Open%20Mirroring/chapter-30.md)
