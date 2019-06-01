---
title: Elastic Search - Getting Start 2
date: 2019-02-12 11:36:15
tags: [architecture, elastic]
---

# 1. Health API

## Health Check

```bash
curl -X GET "http://{{host}}:9200/_cat/health?v"
```

- Green - everything is good (cluster is fully functional)
  - Note: When a cluster is red, it will continue to serve search requests from the available shards but you will likely need to fix it ASAP since there are unassigned shards.
- Yellow - all data is available but some replicas are not yet allocated (cluster is fully functional)
- Red - some data is not available for whatever reason (cluster is partially functional)

<!--more-->

## List Nodes

```bash
curl "http://{{host}}:9200/_cat/nodes?v"
```

# 2. Indices API

## 2.1 List

```bash
curl "http://{{host}}:9200/_cat/indices?v"
```

## 2.2 Create

```bash
curl -X PUT "http://{{host}}:9200/customer?pretty"
curl "http://{{host}}:9200/_cat/indices?v"
```

## 2.3 Delete

```bash
curl -X DELETE "http://{{host}}:9200/customer?pretty"
curl "http://{{host}}:9200/_cat/indices?v"
```

# 3. Document API

## 3.1 Query

```bash
curl "http://{{host}}:9200/movies/movie/1"

curl "http://{{host}}:9200/movies/_search?q=Dark"
```

have another posts talk about **query**

## 3.2 Create

```bash
curl -X PUT "http://{{host}}:9200/movies/movie/109487" -H 'Content-Type: application/json' -d '{ "genre": ["IMAX", "Sci-Fi"], "title": "Interstellar", "year": 2014 }'
curl "http://{{host}}:9200/movies/movie/109487"
```

## 3.3 Update

```bash
curl -X POST "http://{{host}}:9200/movies/movie/109487/_update" -H 'Content-Type: application/json' -d '{ "doc": { "title": "Interstellar" } }'
```

Every document has a _version field
Elasticsearch documents are immutable.

- a new document is created with an incremented _version
- the old doc is marked for deletion

## 3.4 Delete

```bash
curl -X DELETE "http://{{host}}:9200/movies/movie/109487"
```
