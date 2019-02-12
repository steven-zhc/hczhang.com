---
title: Elastic Search 1
date: 2018-10-10 17:39:35
tags: [architecture, elastic]
---

# Overview

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

# Goals

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

# Install

- Elastic Search
- Kibana
  - http://localhost:5601

# Terms

## Compare to RDBMS

| Relationship DB | Elasticsearch |
| - | - |
| database | index |
| table | type |
| row | document |
| column | field |

## Index

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

## Type

> A Representation of a class of similar Documents

For example

`index/type/document`

One type per index

## Document

> A Document in Elasticsearch is an Individual Entry that is the Primary method for adding data.

## Field

> A Field is an Individual Entry in an Elasticsearch Document

## Example

| Object | Elasticsearch |
| - | - |
| Entertainment | index |
| Movies | type |
| Movie | document |
| TITLE :: "text" | field |
| RATING :: "keyword" | field |
| ACTOR_COUNT :: "int" | field |

### Kibana

```json
PUT my_movies
{
    "mappings": {
        "movie": {
            "properties": {
                "name": {
                    "type":"text"
                },
                "actor_count": {
                    "type":"integer"
                },
                "date": {
                    "type":"date"
                }
            }
        }
    }
}
```

```json
PUT /my_moviews/movie/1/_create
{
    "name": "Example Movie One",
    "actor_count": 10,
    "date": "2012-06-09"
}
```

```http
DELETE /my_moviews/movie/1
```

# Shards and Replicas

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

# Bulk API

Allows you to index multiple documents at one time.

```
POST my_movies/_bulk
{"index":{"_index":"my_movies","_type":"movie","_id":"2"}}
{"name":"Sample Movie 2","actor_count":9,"date":"2015-01-10"}
```

# REST Queries

| DB | Verb |
| - | - |
| Create | POST |
| Read | GET |
Update | POST (partial) / PUT (whole)
Delete | Delete

`curl -X <VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

```
curl -XGET http://localhost:9200/INDEX_NAME/_search?q=name:John

# Shorthand version
GET /INDEX_NAME/_search?q=name:John
```
