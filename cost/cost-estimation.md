# Cost estimation – Amazon OpenSearch Service

Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgül, Nathan Stampfli  
Date: June 7, 2026

---

## Scenario

A Swiss SME runs a small e-commerce web application on AWS (region: eu-west-1, Ireland).
The application produces logs from four components: web server, checkout service, payment service, and load balancer.

**Assumptions:**

| Parameter | Value |
| --- | --- |
| Log volume | 5 GB/day |
| Retention period | 30 days |
| Replication | 1 replica |
| Deployment | Managed cluster (not serverless) |
| Region | eu-west-1 (Ireland) |

---

## Storage calculation

```text
Raw log volume:     5 GB/day x 30 days     = 150 GB
OpenSearch overhead (indexing ~10% extra):  +15 GB
Replica copy (x1):                          x2
────────────────────────────────────────────────────
Total storage needed:                      ~330 GB
```

We provision **400 GB EBS (gp3)** to have some headroom.

---

## Cluster sizing

For a small SME with low-to-moderate query load, a minimal but functional cluster:

| Node | Type | Count | Purpose |
| --- | --- | --- | --- |
| Data nodes | `t3.small.search` | 2 | Store and query data |
| Dedicated master | none | 0 | Not needed below 10 nodes |

> A single `t3.small.search` has 2 vCPU, 2 GB RAM — sufficient for 5 GB/day ingest at this scale.

---

## Monthly cost breakdown (eu-west-1, June 2026)

| Component | Calculation | Monthly cost |
| --- | --- | --- |
| Instance hours (2x t3.small.search) | 2 x $0.036/h x 730 h | **$52.56** |
| EBS storage (400 GB gp3) | 400 GB x $0.122/GB | **$48.80** |
| Snapshots in S3 (~20 GB) | 20 GB x $0.023/GB | **$0.46** |
| Inbound data transfer | Free on AWS | **$0.00** |
| OpenSearch Dashboards | Included | **$0.00** |
| **Total** | | **~$102/month** |

> Prices from the [AWS pricing page](https://aws.amazon.com/opensearch-service/pricing/) as of June 2026. Prices may vary.

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

- Use a **managed cluster** with 2 `t3.small.search` nodes — ~$100/month
- Serverless is not cost-effective at this scale
- Monitor storage monthly and adjust retention period to control cost
- Avoid over-provisioning: start small, scale up when needed
