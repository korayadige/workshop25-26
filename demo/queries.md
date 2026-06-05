# OpenSearch demo queries

## Setup

Start the environment:

```bash
docker compose up -d
```

Wait ~30 seconds, then open OpenSearch Dashboards at <http://localhost:5601>  
No login required (security is disabled for the local demo).

Create the index and load sample data via the Dev Tools console (Menu → Dev Tools):

```bash
PUT /logs
```

Then bulk-insert the sample logs:

```bash
POST /_bulk
<paste content of sample-logs.json here>
```

---

## Step 1 — Search all logs

```json
GET /logs/_search
{
  "query": {
    "match_all": {}
  }
}
```

Expected: returns all 15 log documents.

---

## Step 2 — Filter ERROR logs only

```json
GET /logs/_search
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}
```

Expected: returns only documents where `level` is `ERROR`.

---

## Step 3 — Filter HTTP 500+ errors

```json
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

Expected: returns documents with status codes 500, 502, 503, 504.

---

## Step 4 — Find slow requests (response time > 1000 ms)

```json
GET /logs/_search
{
  "query": {
    "range": {
      "responseTimeMs": {
        "gte": 1000
      }
    }
  }
}
```

Expected: returns the payment timeouts and checkout errors that took several seconds.

---

## Step 5 — Combine filters: ERROR + slow response

```json
GET /logs/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" } },
        { "range": { "responseTimeMs": { "gte": 1000 } } }
      ]
    }
  }
}
```

Expected: only errors that also had a high response time — the most critical incidents.

---

## Step 6 — Count errors per service (aggregation)

```json
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

Expected: shows which service caused the most errors (`checkout` should be highest).

---

## Step 7 — Average response time per service

```json
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

Expected: shows average latency per service — useful for performance monitoring.
