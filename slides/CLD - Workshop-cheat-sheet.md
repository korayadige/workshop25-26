# Amazon OpenSearch Service cheat sheet

Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgul, Nathan Stampfli, Zweifel Abram, Victor Giordnai
Date: June 7, 2026

## Why Amazon OpenSearch Service?

Amazon OpenSearch Service is useful for a Swiss SME that needs to centralize, search and analyze logs from several applications and servers.

In this example, a small e-commerce company running a web application on AWS may have logs distributed across EC2 instances, load balancers and application services. Without a centralized log search system, troubleshooting HTTP 500 errors, slow requests or suspicious activity is difficult.

OpenSearch indexes these logs and makes them searchable. It also provides dashboards and aggregations to analyze errors, traffic and application behavior.

### Typical inputs

- Application logs
- Web server logs
- Security logs
- JSON documents
- CloudWatch logs

### Typical outputs

- Search results
- Dashboards
- Error statistics
- Filtered logs
- Alerts

---

## Benefits and limitations

### Benefits

- **Managed service:** AWS manages much of the infrastructure, which reduces operational work for the company.
- **Fast search:** Logs and JSON documents are indexed, so searching is much faster than manually reading log files.
- **Centralized observability:** Logs from different servers and applications can be analyzed in one place.
- **Dashboards:** OpenSearch Dashboards can be used to visualize errors, traffic, latency or security events.
- **AWS integration:** It can be integrated with services such as CloudWatch, S3, Lambda or Kinesis.
- **Scalability:** The cluster can be adapted to the volume of data and the number of queries.
- **Security features:** Access control, encryption and VPC deployment can be configured.

### Limitations

- Cost can increase quickly if the company stores too much data or keeps logs for too long.
- Cluster sizing is not trivial: wrong instance types, too many shards or insufficient storage can reduce performance.
- Vendor dependency: using the managed AWS service creates dependency on AWS infrastructure, pricing and service limits.
- Production usage still requires monitoring, tuning and security configuration.

---

## Cost structure

**Amazon OpenSearch Service can be used in two main deployment models: managed clusters and serverless collections.**

For managed clusters, the main cost components are:

- Instance hours: the compute instances used by the OpenSearch cluster.
- Storage: EBS volumes used to store indexed data.
- Data transfer: network traffic in and out of the service.
- Snapshots and backups: storage used for backups.
- Optional ingestion pipelines: if OpenSearch Ingestion is used to process and send data into OpenSearch.

For OpenSearch Serverless, the main cost components are:

- Compute capacity measured in OpenSearch Compute Units (OCUs).
- Storage used by indexed data.
- Ingestion and query capacity.

A cost estimate depends on: daily log volume, retention period, replication factor, number and type of nodes, storage size, query frequency.

### Monthly cost example

A Swiss SME produces approximately 5 GB of logs per day and wants to keep logs for 30 days.

```text
5 GB/day x 30 days = 150 GB raw
+ indexing overhead + 1 replica = ~330 GB stored
Provisioned: 400 GB EBS (gp3)
```

**Monthly cost (eu-west-1, managed cluster):**

| Component | Monthly cost |
| --- | --- |
| 2x `t3.small.search` instances | $52.56 |
| 400 GB EBS storage (gp3) | $48.80 |
| Snapshots (S3, ~20 GB) | $0.46 |
| **Total** | **~$102/month** |

A production setup (3 master + 3 data nodes, 1 TB storage) costs **$600–900/month**.

OpenSearch Serverless starts at ~$700/month minimum due to always-on OCUs — not cost-effective for a small steady workload.

---

## How to get started

### Prerequisites

To start using Amazon OpenSearch Service, the company needs:

- an AWS account
- IAM permissions to create and manage OpenSearch resources
- basic knowledge of JSON and HTTP requests
- a source of data, such as application logs or sample JSON documents
- optional tools such as AWS CLI, curl, Postman or Python
- for production: VPC, security groups, encryption and access control

### Preparation steps

1. Open the AWS Management Console.
2. Go to Amazon OpenSearch Service.
3. Create an OpenSearch domain or a serverless collection.
4. Choose the deployment type: managed cluster for more control, or serverless for simplified capacity management.
5. Configure network access: public for a test environment, VPC for production.
6. Configure authentication and permissions.
7. Create an index.
8. Send a first JSON document.
9. Run a search query.
10. Open OpenSearch Dashboards to visualize the data.

```text
AWS Console → OpenSearch Service → Create domain
→ Network + permissions → Create index → Insert document
→ Search → Dashboard
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
    "match": {
      "name": "chocolate"
    }
  }
}
```

OpenSearch returns the document where the field `name` matches the word `chocolate`.

---

## Common commands / operations / configurations / usage patterns

### Create an index

An index is a logical place where documents are stored.

```bash
PUT /logs
```

This creates an index named `logs`.

---

### Add a log document

A log entry can be stored as a JSON document.

```bash
POST /logs/_doc
{
  "timestamp": "2026-06-07T10:00:00Z",
  "level": "ERROR",
  "service": "checkout",
  "statusCode": 504,
  "message": "Payment service timeout"
}
```

---

### Search all documents

```bash
GET /logs/_search
{
  "query": {
    "match_all": {}
  }
}
```

This returns all documents from the `logs` index.

---

### Search error logs

```bash
GET /logs/_search
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}
```

This returns log entries where the field `level` contains `ERROR`.

---

### Filter logs by time range

```bash
GET /logs/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}
```

This returns logs from the last hour.

---

### Filter HTTP server errors

```bash
GET /logs/_search
{
  "query": {
    "range": {
      "statusCode": {
        "gte": 500
      }
    }
  }
}
```

This returns HTTP server errors such as 500, 502, 503 or 504.

---

### Aggregate errors by service

```bash
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "service.keyword"
      }
    }
  }
}
```

This counts how many errors occurred for each service.

---

### Typical usage patterns

Amazon OpenSearch Service is commonly used for:

- centralized log search
- application monitoring
- security event analysis
- full-text search
- error investigation
- dashboard creation
- alerting on abnormal behavior
- analyzing HTTP status codes and latency
- troubleshooting distributed applications
