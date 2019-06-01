---
title: Elastic Search - Getting Start 1
date: 2018-10-10 17:39:35
tags: [architecture, elastic]
---

# 1. Overview

## Upside

- Easy to setup
- Abstracts away low level
- Scales beautifully
- Feature rich (out of box)

## Downside

- Poorly managed indeces
- Inefficient queries
- Web facing clasters
- Used as primary data store

> Elasticsearch is Robust, Highly Available, Distributed Search and Analytics Engine.

<!--more-->

# 2. Goals

- Lightning Fast Search
  - Scalable
  - Highly avaliable
  - Distributed. no Single Point failure
- Analytics Engine
  - Aggregations
  - Log analysis
  - Geo-location data
  - Machine learning
- Near Real-Time (NRT)
  - Add Doc -> Inverted Index -> Available for search
- Powerful Rest API
  - Search API: localhost:9000/index_search?q=*&pretty
  - DSL

# 3. Install

```yaml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.1
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
  kibana:
    image: docker.elastic.co/kibana/kibana:6.7.1
    ports:
      - "5601:5601"
```

# 4. Terms

## 4.1 Compare to RDBMS

| Relationship DB | Elasticsearch |
| - | - |
| database | index |
| table | type |
| row | document |
| column | field |

## 4.2 Index

> An Index is a Logical Namespace that points to 1 or more Shards in an Elasticsearch Cluster

Think about *Shards* as disk partition, or container of data.
Index is where data are stored in form of document

- Index is broken into shards
- Shards are containers for data

```bash
# Create a new index
http PUT http://slc10vfs:9200/my_test_index

# Get index info
http http://slc10vfs:9200/my_test_index?pretty

# Delete an index
http DELETE http://slc10vfs:9200/my_test_index
```

## 4.3 Type

> A Representation of a class of similar Documents

For example

`index/type/document`

One type per index

## 4.4 Document

> A Document in Elasticsearch is an Individual Entry that is the Primary method for adding data.

## 4.5 Field

> A Field is an Individual Entry in an Elasticsearch Document

## 4.6 Example

| Object | Elasticsearch |
| - | - |
| Movies | index |
| Movie | type |
| Row   | Document  |
| TITLE :: "text" | field |
| RATING :: "keyword" | field |
| ACTOR_COUNT :: "int" | field |

## 4.7 Mapping

A mapping is a schema definition

- field types
  - text, keyword, byte, short, integer, long, float, double, boolean, date
- field index
  - do you want this field to be queryable? ture/false
- field analyzer
  - define our tokenizer and token filter. standard / whitespace / simple/ english
    - characher filters: remove html encoding convert & to and
    - tokenizer: split strings on whitespace / punctuation/ non-letter
    - token filter: lowercasing, stemming, synonyms, stopwords
    - standard: splits on word boundaries
    - simple: splits on anything isn't a letter, and lowercases
    - whitespace: splits on whitespace but doesn't lowercase
    - language: i.e. engligh. accounts for language-specific stopwords and stemming

### analyzer

Sometimes text fields should be exact-match

- use **keyword** mapping type to suppress analyzing (eact match only)
- use **text** type to allow analyzing

search on analyzed field will return anything remotely relevent

```json
PUT my_movies
{
    "mappings": {
        "movie": {
            "properties": {
                "id": { "type":"text" },
                "year": { "type":"date" },
                "genre": { "type": "keyword" },
                "title": { "type": "text", "analyzer": "english"}
            }
        }
    }
}

curl http://{{host}}:9200/movies/_mapping/movie
```

```json
PUT /my_moviews/movie/1
{
    "name": "Example Movie One",
    "actor_count": 10,
    "date": "2012-06-09"
}
```

```http
DELETE /my_moviews/movie/1
```

# 5. Shards and Replicas

## Shards

- Default: Five shards per index
- Make Elasticsearch distributed
- Auto-balanced by failover

## Replicas

> Replicas are Duplications of Primary Shards

- Takes over if primary fails
- Node rejoins after failure
- New node asynchronizes with others

## Example

2 nodes, 5 shards and 1 replica
N = Primary shard
R = Replica shard

```
Cluster status: GREEN      Clauster status: YELLOW     Cluster status: GREEN
Node 1                     Node 2                      Node 2
[ N0 N1 N2 R3 R4 ]         [ N0 N1 N2 R3 R4 ]          [ N0 N1 N2 R3 R4 ]
                      =>                          =>
Node 2                     Node 2 (FAILED)             Node 2 (STARTED & SYNCED)
[ R0 R1 R2 N3 N4 ]                                     [ R0 R1 R2 N3 N4 ]
```

```HTTP
PUT test_index
{
    "settings": {
        "number_of_shards": 1
    }
}

PUT test_index/_settings
{
    "number_of_replicas": 3
}

```

- Cannot set shards after created
- But could change replicas

# 6. Bulk API

Allows you to index multiple documents at one time.

```
POST my_movies/_bulk
{"index":{"_index":"my_movies","_type":"movie","_id":"2"}}
{"name":"Sample Movie 2","actor_count":9,"date":"2015-01-10"}
```

```bash
curl -XPUT 127.0.0.1:9200/_bulk --data-binary @movies.json

$ cat movies.json
{ "create" : { "_index" : "movies", "_type" : "movie", "_id" : "135569" } }
{ "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
```

# 7. REST Queries

| DB | Verb |
| - | - |
| Create | POST |
| Read | GET |
| Update | POST (partial) / PUT (whole)
| Delete | Delete

`curl -X <VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

```
curl -XGET http://localhost:9200/INDEX_NAME/_search?q=name:John

# Shorthand version
# could be runnable in kibana
GET /INDEX_NAME/_search?q=name:John
```
