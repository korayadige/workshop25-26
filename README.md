
# CLD Workshop – Amazon OpenSearch Service

Course: HEIG-VD CLD 2025/26  
Authors: Koray Akgul, Nathan Stampfli, Zweifel Abram, Victor Giordnai

Topic: Amazon OpenSearch Service / Elasticsearch  
Use case: Centralized log search and analysis for a Swiss SME

## Project overview

This repository contains the material for a workshop about Amazon OpenSearch Service.

The goal is to explain how a Swiss small or medium-sized enterprise can use Amazon OpenSearch Service to centralize, search, analyze and visualize logs coming from several applications or servers.

The project includes:

- a cheat sheet
- a proof-of-concept demo
- common OpenSearch queries
- a cost estimation
- a presentation plan

## Scenario

A Swiss SME runs a small e-commerce web application on AWS.  
The application produces logs from different components such as:

- web servers
- application services
- load balancers
- security systems

Without a centralized log search system, troubleshooting errors such as HTTP 500 or 504 responses is slow and difficult.

Amazon OpenSearch Service solves this problem by indexing logs and making them searchable. It also provides dashboards and aggregations to analyze errors, traffic and application behavior.

## Repository structure

```text
.
├── README.md
├── CLD-Workshop-cheat-sheet.md
├── demo/
│   ├── docker-compose.yml
│   ├── sample-logs.json
│   └── queries.md
├── slides/
│   └── presentation-plan.md
└── cost/
    └── cost-estimation.md
```

## Main concepts

The workshop covers the following concepts:

OpenSearch Service
Elasticsearch / OpenSearch
index
document
field
query
aggregation
dashboard
cluster
node
shard
replica
log analysis
cost structure
vendor lock-in
Demo idea

## The demo shows a small log analysis scenario:

Start OpenSearch and OpenSearch Dashboards.
Insert sample log documents.
Search all logs.
Filter error logs.
Filter HTTP 500+ errors.
Aggregate errors by service.
Explain how this maps to Amazon OpenSearch Service on AWS.
Files
cheat-sheet.md: short technical summary of Amazon OpenSearch Service
demo/sample-logs.json: example log documents
demo/queries.md: useful OpenSearch queries
cost/cost-estimation.md: monthly cost scenario
slides/presentation-plan.md: structure for the oral presentation


