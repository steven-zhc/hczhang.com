---
title: Elastic Search - Getting Start 2
date: 2019-02-12 11:36:15
tags: [architecture, elastic]
---

# Health API

Health Check
```bash
curl -X GET "http://{{host}}:9200/_cat/health?v"
```

- Green - everything is good (cluster is fully functional)
  - Note: When a cluster is red, it will continue to serve search requests from the available shards but you will likely need to fix it ASAP since there are unassigned shards.
- Yellow - all data is available but some replicas are not yet allocated (cluster is fully functional)
- Red - some data is not available for whatever reason (cluster is partially functional)

<!--more-->

List Nodes

```bash
curl "http://{{host}}:9200/_cat/nodes?v"
```

# Indices API

## List

```bash
curl "http://{{host}}:9200/_cat/indices?v"
```

## Create

```bash
curl -X PUT "http://{{host}}:9200/customer?pretty"
curl "http://{{host}}:9200/_cat/indices?v"
```

## Delete

```bash
curl -X DELETE "http://{{host}}:9200/customer?pretty"
curl "http://{{host}}:9200/_cat/indices?v"
```

# Document API

## Query

```bash
curl "http://{{host}}:9200/customer/_doc/1?pretty"
```

## Create and Update Index

```bash
curl -X PUT "http://{{host}}:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d '{ "name": "John Doe" }'
curl "http://{{host}}:9200/customer/_doc/1?pretty"
```

Create with generated ID

```bash
curl -X POST "http://{{host}}:9200/customer/_doc?pretty" -H 'Content-Type: application/json' -d '{ "name": "Jane Doe" }'
```

## Update

```bash
curl -X POST "http://{{host}}:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d '{ "doc": { "name": "Jane Doe", "age": 20 } }'
```

## Delete

```bash
curl -X DELETE "http://{{host}}:9200/customer/_doc/2?pretty"
```