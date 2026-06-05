# Cost estimation – Amazon OpenSearch Service

Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgül, Nathan Stampfli  
Date: June 7, 2026

---

## Scenario

A Swiss SME runs a small e-commerce web application on AWS.
The application produces logs from four components: web server, checkout service, payment service, and load balancer.

**Assumptions:**

| Parameter | Value |
| --- | --- |
| Log volume | 5 GB/day |
| Retention period | 30 days |
| Replication | 1 replica |
| Deployment | Managed cluster (not serverless) |
| Region | EU (Zurich) — data stays in Switzerland |

---

## Storage calculation

```text
Raw log volume:     5 GB/day x 30 days     = 150 GB
OpenSearch overhead (indexing ~10% extra):  +15 GB
Replica copy (x1):                          x2
────────────────────────────────────────────────────
Total storage needed:                      ~330 GB
```

We provision **2x 150 GB EBS gp3** (one per node) = 300 GB total.

---

## Cluster sizing

For a small SME with low-to-moderate query load, a minimal but production-ready cluster:

| Node | Type | Count | Purpose |
| --- | --- | --- | --- |
| Data nodes | `t3.medium.search` | 2 | Store and query data |
| Dedicated master | none | 0 | Not needed below 10 nodes |

> A single `t3.medium.search` has 2 vCPU, 4 GB RAM — sufficient for 5 GB/day ingest and dashboard queries at this scale. `t3.small` (2 GB RAM) is too tight for concurrent search and indexing in practice.

---

## Monthly cost breakdown (EU Zurich, June 2026)

Region: **EU (Zurich)** — data stays in Switzerland, required for GDPR and Swiss nDSG compliance.  
Configuration: 2 data nodes, 2x 150 GB gp3 EBS, no dedicated master, no UltraWarm.

| Scenario | Instance | RAM | USD/month | CHF/month |
| --- | --- | --- | --- | --- |
| Dev / test | `t3.small.search` | 2 GB | **$119.36** | **~107 CHF** |
| SME production | `t3.medium.search` | 4 GB | **$186.52** | **~168 CHF** |

> Source: AWS Pricing Calculator, EU (Zurich), June 2026.

**Why we chose `t3.medium.search`:** At 5 GB/day with concurrent log ingestion, search queries, and OpenSearch Dashboards running simultaneously, `t3.small.search` (2 GB RAM) is too tight. `t3.medium.search` (4 GB RAM) provides enough headroom for stable production use at an acceptable cost of ~168 CHF/month.

---

## What drives cost up

| Factor | Impact |
| --- | --- |
| Longer retention (e.g. 90 days) | Storage triples — +$100/month |
| More or larger nodes | Biggest cost driver |
| Multiple replicas | Doubles storage |
| High query volume | May require larger instance types |
| OpenSearch Ingestion pipeline | Additional OCU cost |

---

## Production scenario (comparison)

A production-grade setup with high availability would require:

| Component | Value |
| --- | --- |
| Dedicated master nodes | 3x `m5.large.search` |
| Data nodes | 3x `r6g.large.search` |
| Storage | 1 TB EBS per data node |
| Replicas | 1 replica |

Estimated cost: **$600–900/month**

This is still far cheaper than running and managing equivalent self-hosted Elasticsearch infrastructure.

---

## Serverless alternative

Amazon OpenSearch Serverless removes the need to size a cluster manually.
Cost is based on **OpenSearch Compute Units (OCUs)**:

- Minimum: 2 OCUs indexing + 2 OCUs search = 4 OCUs always-on
- Price: ~$0.24/OCU/hour — 4 x $0.24 x 730 = **~$700/month minimum**

**Serverless is more expensive for a small steady workload**, but cost-effective for highly variable or unpredictable traffic where a managed cluster would need to be over-provisioned.

---

## Recommendation

For a Swiss SME with ~5 GB/day log volume:

- Use a **managed cluster** with 2 `t3.medium.search` nodes in EU (Zurich) — ~168 CHF/month
- Serverless is not cost-effective at this scale
- Monitor storage monthly and adjust retention period to control cost
- Set index lifecycle management (ILM) rules to automatically delete old logs
