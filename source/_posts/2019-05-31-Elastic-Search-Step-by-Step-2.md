---
title: Elastic Search Step by Step 2
date: 2019-05-31 10:54:36
tags: [architecture, elastic]
---

# Searching with elasticsearch

This section will focus on query

<!--more-->

## query lite

URI Search

- /movies/movie/_search?q=title:star
- /movies/movie/_search?q=+year:>2010+title:trek

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?q=title:star&pretty

# becareful about url encoding
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?q=%2Byear%3A%3E2010%2Btitle%3Atrek
```

## request body search

query DSL is in the request body as JSON

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty -d `
{
    "query": {
        "match": {
            "title": "star"
        }
    }
}
`
```

- **filters** ask a yes/no question of your data
- **queries** return data in terms of relevance

use filters when you can - they are faster and cacheable.

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "bool": {
            "must": { "term": {"title": "trek" }},
            "filter": {"range": {"year": {"gte": 2010}}}
        }
    }
}'
```

### some types of filters

filters are wrapped in a *filter: {}* block

could combine filters inside queries, or queries inside filters

- term: filter by exact values
  - {"term": {"year": 2014"}}
- terms: match if any exact values in a list match
  - {"terms": {"genre": ["Sci-Fi", "Adventure"]}}
- range: Find numbers or dates in a given range(gt, gte, lt, lte)
  - {"range": {"year": {"gte": 2010}}}
- exists: Find doc where a field exists
  - {"exists": {"field": "tags"}}
- missing: Find doc where a field is missing
  - {"missing": {field": "tags"}}
- bool: Combine filters with Boolean logic (must, must_not, should)

### some types of queries

queries are wrapped in a *query: {}* block

could combine filters inside queries, or queries inside filters

- match_all: return all documents and is the default. Normally used with a filter.
  - {"match_all": {}}
- match: searches analyzed results, such as full text search.
  - {"match": {"title": "star"}}
- multi-match: run the same query on multiple fields.
  - {"multi_match": {"query": "star", "fields": ["title", "synopsis"]}}
- bool: Works like a bool filter, but results are scored by relevance.
- match_phase: must find all terms, in the right order
  - {"match_phase": {"title": "star wars"}}

## Phrase search

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phase": {"title": "star wars"}
    }
}'
```

## slop

order matters, but you're OK with some words being in between the terms.
```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phase": {"title": "star beyond", "slop": 1}
    }
}'
```

"quick brown fox" would match "quick fox" with a slop of 1

## proximity queries

> Remember this: query results are sorted by relevance.

Just use a really high slop for proximity queries

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty -d '
{
    "query": {
        "match_phase": {"title": "star beyond", "slop": 100}
    }
}'
```