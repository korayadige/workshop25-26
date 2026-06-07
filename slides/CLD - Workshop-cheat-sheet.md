# Amazon OpenSearch Service cheat sheet


Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgul,  Nathan Stampfli,  Zweifel Abram,    Victor Giordani  
Date: June 8, 2026


## Why Amazon OpenSearch Service?

Amazon OpenSearch Service is highly useful for a Swiss SME that needs to centralize, search, and analyze logs from several applications and servers in real time.

In our scenario, a small e-commerce company running a web application on AWS has logs distributed across multiple EC2 instances, load balancers, and internal application services. Without a centralized log search system, troubleshooting critical production errors — such as HTTP 500 internal errors, slow database requests, or suspicious security activity — is extremely slow and difficult because engineers must manually log into individual servers to inspect text files.

OpenSearch solves this problem by collecting, parsing, and indexing these logs automatically, making them instantly searchable through an API or a visual interface. It provides dashboards and powerful aggregations to analyze system errors, traffic spikes, and application behavior at a glance.

- **Typical Inputs:** Application logs, web server logs (Nginx/Apache), security/firewall logs, CloudWatch logs, and structured JSON documents.
- **Typical Outputs:** Sub-second search results, interactive dashboards, error statistics/graphs, filtered real-time log streams, and automated Slack/Email alerts.

---

## Benefits and limitations

### Benefits

- **Managed Service:** AWS handles infrastructure provisioning, software patching, failure recovery, and automated backups, which drastically reduces operational work for a small IT team.
- **Sub-Second Search Latency:** Logs and JSON documents are indexed using an inverted index, meaning searching through millions of lines is far faster than manually reading or using `grep` on log files.
- **Centralized Observability:** Logs from different servers, cloud services, and regions can be correlated and analyzed in one single place.
- **Rich Dashboards:** OpenSearch Dashboards comes built-in to visualize errors, traffic trends, system latency, or security events for developers and business managers alike.
- **AWS Integration:** Native integration with CloudWatch, S3, Kinesis, and Lambda.

### Limitations

- **No Scale-to-Zero (managed clusters):** A managed OpenSearch cluster requires dedicated instances running 24/7. Even with zero traffic at night, you pay for the baseline infrastructure. The new Serverless Next Generation (GA May 2026) does support Scale-to-Zero, but only helps when traffic genuinely drops to zero.
- **Storage Cost and Retention:** Cost increases rapidly if the company stores too much raw data or keeps logs for too long without lifecycle management rules.
- **Cluster Sizing Complexity:** Wrong instance types, too many shards, or insufficient storage can significantly reduce performance. Sizing requires experience.
- **Vendor Dependency:** Relying on the managed AWS service creates a strong dependency on AWS infrastructure, pricing, and service limits — see Vendor Lock-in below.

---

## Cost structure

Amazon OpenSearch Service can be deployed in two main models: **Managed Clusters** (dedicated instances) and **Serverless Collections** (auto-scaling capacity).

For a managed cluster, the main cost components are:

1. **Instance Hours:** Compute instances used by data nodes and cluster manager nodes.
2. **Storage:** EBS volumes (gp3 SSD) used to store active indexed data.
3. **Data Transfer:** Network traffic in/out of the service or across Availability Zones.

### Monthly cost example (Swiss SME scenario)

- **Scenario:** 5 GB of raw logs per day, kept searchable for 30 days.
- **Data estimate:** 5 GB/day x 30 days = 150 GB. With 1 replica: **300 GB total storage**.
- **Cluster:** 2 x `t3.medium.search` nodes (2 vCPU, 4 GB RAM each), 150 GB gp3 per node.

Region: **EU (Zurich)** — data stays in Switzerland (GDPR + Swiss nDSG compliance).  
Configuration: 2 nodes, 2x 150 GB gp3 EBS, no dedicated master.

| Scenario | Instance | USD/month | CHF/month |
| --- | --- | --- | --- |
| Dev / test | `t3.small.search` | $119.36 | ~107 CHF |
| SME production | `t3.medium.search` | $186.52 | ~168 CHF |

**Serverless alternative (Next Generation — GA May 28, 2026):** AWS announced the next generation of OpenSearch Serverless with true Scale-to-Zero: the cluster scales down to 0 OCU when idle, eliminating the previous ~$700/month minimum. AWS claims up to 60% cost savings vs. over-provisioned managed clusters, with resource creation in seconds and 20× faster auto-scaling than the previous generation. However, Scale-to-Zero only helps when traffic drops to zero. For a Swiss SME with a continuous 24/7 log stream (5 GB/day), the managed cluster at ~168 CHF/month remains more predictable and cost-effective.

**Production setup** (3 master + 3 data nodes, 1 TB storage): **$600–900/month**.

---

## Vendor lock-in

OpenSearch is open-source (Apache 2.0 license), so the query language and APIs are not proprietary. However, using Amazon OpenSearch Service still creates dependency on:

- AWS IAM for authentication and access control.
- AWS-specific ingestion pipelines (OpenSearch Ingestion, Kinesis).
- CloudWatch integration and VPC networking configuration.

**Migration risk:** Moving to a self-hosted cluster or another provider requires re-exporting data, reconfiguring pipelines, and adapting IAM policies — costly in time and money.

**Recommendation:** The lock-in is acceptable for most Swiss SMEs given the operational savings. Consider self-hosting only if AWS costs become prohibitive or if data sovereignty regulations require it.

---

## How to get started

### Prerequisites

- An active AWS account with IAM permissions to create OpenSearch domains.
- Docker and Docker Compose installed locally (for testing before cloud deployment).
- A tool to send HTTP commands: `curl`, Postman, or the OpenSearch Dashboards Dev Tools console.

### Preparation steps

```text
AWS Console → Amazon OpenSearch Service → Create domain
→ Configure Network/VPC → Configure Security/Auth
→ Deploy cluster → Create index → Insert document → Search
```

For a local development environment:

```bash
docker compose up -d
curl http://localhost:9200
```

### Hello-world usage example

Create an index:

```bash
PUT /products
```

Add a document:

```bash
PUT /products/_doc/1
{
  "name": "Swiss chocolate",
  "category": "food",
  "price": 4.50
}
```

Search the document:

```bash
GET /products/_search
{
  "query": {
    "match": { "name": "chocolate" }
  }
}
```

OpenSearch returns the document where the field `name` matches `chocolate`.

---

## Common commands / operations / configurations / usage patterns

OpenSearch relies on a RESTful API using JSON bodies. Below are the core commands for log analysis.

### 1. Create a logs index with mappings

```bash
PUT /logs
{
  "mappings": {
    "properties": {
      "timestamp":      { "type": "date" },
      "level":          { "type": "text" },
      "service":        { "type": "text" },
      "statusCode":     { "type": "integer" },
      "responseTimeMs": { "type": "integer" },
      "message":        { "type": "text" }
    }
  }
}
```

### 2. Ingest a log document

```bash
POST /logs/_doc
{
  "timestamp": "2026-06-07T10:00:00Z",
  "level": "ERROR",
  "service": "checkout-backend",
  "statusCode": 504,
  "message": "Payment gateway timeout to PostFinance API"
}
```

### 3. Search all logs

```bash
GET /logs/_search
{ "query": { "match_all": {} } }
```

### 4. Filter HTTP server errors (500+)

```bash
GET /logs/_search
{
  "query": {
    "range": { "statusCode": { "gte": 500 } }
  }
}
```

### 5. Filter logs by time range

```bash
GET /logs/_search
{
  "query": {
    "range": { "timestamp": { "gte": "now-1h", "lte": "now" } }
  }
}
```

### 6. Aggregate errors by service

```bash
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "errors_by_service": {
      "filter": { "match": { "level": "ERROR" } },
      "aggs": {
        "by_service": {
          "terms": { "field": "service.keyword" }
        }
      }
    }
  }
}
```

### 7. Average response time per service

```bash
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "avg_response_by_service": {
      "terms": { "field": "service.keyword" },
      "aggs": {
        "avg_response": {
          "avg": { "field": "responseTimeMs" }
        }
      }
    }
  }
}
```

---

## When to use / when not to use

**Use Amazon OpenSearch Service when:**

- You need centralized log search across multiple services or servers.
- You need full-text search on large volumes of JSON data.
- You want dashboards and alerting without building them from scratch.
- You are already on AWS and want native integration.

**Do not use it when:**

- You need a transactional database — use RDS or DynamoDB instead.
- Your log volume is small and CloudWatch Logs Insights is sufficient.
- You cannot justify the ~$100/month minimum for a small steady workload.
- You need very low-latency key-value lookups — use ElastiCache instead.

---

## References

- [Amazon OpenSearch Service documentation](https://docs.aws.amazon.com/opensearch-service/)
- [OpenSearch project (open-source)](https://opensearch.org/)
- [AWS pricing page](https://aws.amazon.com/opensearch-service/pricing/)
- [OpenSearch Dashboards guide](https://opensearch.org/docs/latest/dashboards/)
