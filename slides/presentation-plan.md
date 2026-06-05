# Presentation plan – Amazon OpenSearch Service

Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgül, Nathan Stampfli  
Date: June 7, 2026  
Total time: 15 minutes + 3 minutes Q&A

---

## Part 1 – Theory (approx. 8 minutes)

### 1. Introduction (1 min)

- Who are we and what is the scenario?
- A Swiss SME runs an e-commerce application on AWS.
- Problem: logs are scattered across EC2 instances, load balancers, services — troubleshooting HTTP 500 errors is slow.
- Solution: Amazon OpenSearch Service centralizes and indexes all logs.

### 2. What is Amazon OpenSearch Service? (2 min)

- Managed search and analytics engine based on OpenSearch (fork of Elasticsearch).
- AWS handles provisioning, patching, scaling, backups.
- Core concepts: index, document, field, query, aggregation, shard, replica.
- Two deployment modes: managed cluster vs. serverless collection.
- Typical inputs: application logs, web server logs, CloudWatch logs, JSON documents.
- Typical outputs: search results, dashboards, alerts, error statistics.

### 3. Benefits and limitations (2 min)

**Benefits:**

- Fast full-text search on large volumes of logs.
- Centralized observability: one place for all services.
- Built-in dashboards (OpenSearch Dashboards, formerly Kibana).
- Native AWS integration: CloudWatch, S3, Kinesis, Lambda.
- Security: encryption at rest/transit, fine-grained access control, VPC support.

**Limitations (be critical):**

- Cluster sizing is complex: wrong instance types or too many shards hurt performance.
- Cost grows quickly with data volume and retention period.
- Not a relational database — no transactions, no joins.
- Operational overhead remains even in managed mode (monitoring, tuning).

### 4. Cost (2 min)

- Scenario: 5 GB/day, 30-day retention, EU (Zurich) — data stays in Switzerland.
- Compared `t3.small.search` (2 GB RAM, $119/mo) vs `t3.medium.search` (4 GB RAM, $187/mo).
- Chose `t3.medium.search`: 2 GB RAM is too tight for concurrent indexing + search + dashboards.
- **Selected: 2x `t3.medium.search` + 2x 150 GB gp3 = $186.52/month (~168 CHF).**
- Production-grade (HA): **$600–900/month**.
- Serverless: minimum ~$700/month — not suitable for small steady workloads.
- Key cost drivers: retention period, number of nodes, storage, replicas.

### 5. Vendor lock-in (1 min)

- OpenSearch is open-source (Apache 2.0 license) — you can self-host if needed.
- However: managed service ties you to AWS APIs, IAM, VPC, CloudWatch integration.
- Migration is possible but costly: data export, re-indexing, pipeline reconfiguration.
- Alternative: self-hosted OpenSearch on EC2 or Kubernetes — more control, more work.
- Recommendation: acceptable lock-in for most SMEs given the operational savings.

---

## Part 2 – Demo (approx. 5 minutes)

### 6. Proof-of-concept demo (5 min)

**Setup (already running before the presentation):**

```bash
docker compose up -d
# OpenSearch at https://localhost:9200
# Dashboards at http://localhost:5601
```

**Demo steps (shown live in OpenSearch Dashboards Dev Tools):**

1. Show the scenario: e-commerce logs from webserver, checkout, payment, auth.
2. Step 1 — Search all logs: `GET /logs/_search { "query": { "match_all": {} } }`
3. Step 2 — Filter ERROR logs only.
4. Step 3 — Filter HTTP 500+ errors.
5. Step 4 — Find slow requests (responseTimeMs > 1000 ms).
6. Step 5 — Combine: ERROR + slow response (most critical incidents).
7. Step 6 — Aggregation: count errors per service — shows `checkout` is the problem.
8. Step 7 — Average response time per service — performance overview.

**Key message:** in a real AWS setup, these same queries run against hundreds of GB of live logs — same API, same dashboards, fully managed.

---

## Part 3 – Conclusion (approx. 2 minutes)

### 7. When to use / when not to use (1 min)

**Use OpenSearch when:**

- You need centralized log search across multiple services or servers.
- You need full-text search on large volumes of JSON data.
- You want dashboards and alerting without building them from scratch.
- You are already on AWS and want native integration.

**Do not use OpenSearch when:**

- You need a transactional database (use RDS or DynamoDB instead).
- Your log volume is small and a simple CloudWatch setup is sufficient.
- You cannot justify the ~$100/month minimum cost for a test/small workload.
- You need very low latency key-value lookups (use ElastiCache instead).

### 8. References (1 min)

- [Amazon OpenSearch Service documentation](https://docs.aws.amazon.com/opensearch-service/)
- [OpenSearch project (open-source)](https://opensearch.org/)
- [AWS pricing page](https://aws.amazon.com/opensearch-service/pricing/)
- [OpenSearch Dashboards guide](https://opensearch.org/docs/latest/dashboards/)
- [AWS Well-Architected — Operational Excellence](https://aws.amazon.com/architecture/well-architected/)
