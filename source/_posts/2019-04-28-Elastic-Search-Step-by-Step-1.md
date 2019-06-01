---
title: Elastic Search Step by Step 1
date: 2019-04-28 12:41:23
tags: [architecture, elastic]
---
This is follow up Elastic Search - Getting Start Post

More details and step by step instruction

<!--more-->

# Mapping and indexing data

## Mapping
A mapping is a schema definition.

```bash
curl -X PUT http://{{host}}:9200/movies -H 'Content-Type: application/json' -d '
{
    "mappings": {
        "movie": {
            "properties": {
                "year": { "type": "date" }
            }
        }
    }
}

curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/_mapping/movie?pretty
'
```

## Import Document

```bash
curl -X PUT http://{{host}}:9200/movies/movie/109487 -H 'Content-Type: application/json' -d '
{
    "genre": ["IMAX", "Sci-Fi"],
    "title": "Interstella",
    "year": 2014
}'

curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/_search?pretty
```

## Bulk import

```bash
wget http://media.sundog-soft.com/es/movies.json

curl -X PUT http://{{host}}:9200/_bulk?pretty --data-binary @movies.json -H 'Content-Type: application/json'
```

## Update

Every document has a _version field

Elasticsearch documents are immutable.

- a new doc is created with an incremented _version
- the old document is marked for deletion

```bash
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/movie/109487?pretty

curl -X POST http://{{host}}:9200/movies/movie/109487/_update -H 'Content-Type: application/json' -d '
{
    "doc": {
        "title": "instellar"
    }
}
'
```

create doc with same id, it also increase _version field. such as:

```bash
curl -X PUT http://{{host}}:9200/movies/movie/109487 -H 'Content-Type: application/json' -d '
{
    "genre": ["IMAX", "Sci-Fi"],
    "title": "Interstella",
    "year": 2014
}'
```

## Delete

```bash
# first of all, search id
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/movies/_search?q=Dark_

# delete by id
curl -H 'Content-Type: application/json' -X DELETE http://{{host}}:9200/movies/movie/109487?pretty
```

## Concurrency

Optimistic concurrency control

In short, ELK use _version field to control version of data

> For conflict _version: Use retry_on_conflicts+N to automatically retry

```bash
curl -H 'Content-Type: application/json' -X PUT http://{{host}}:9200/movies/movie/109487?version=3 -d '
{
    "genre": ["Imax", "Sci-Fi"],
    "title": "Interstellar Foo",
    "year": 2014,
}
'

# keey to retry if conflict on data
curl -H 'Content-Type: application/json' -X POST http://{{host}}:9200/movies/movie/109487/_update?retry_on_conflict=5 -d '
{
    "doc": {
        "title": "Interstellar"
    }
}
'
```

## Data modeling

### Normalized data *VS* Denormalized data

> think about Denormalized data in ELK

### Strategies for relational data

In short answer, *join* field (ELK 6)

parent and child data should be store in same shards

```bash
curl -H 'Content-Type: application/json' -X PUT http://{{host}}:9200/series -d '
{
    "mappings": {
        "movie": {
            "properties": {`
                "film_to_franchise": {
                    "type": "join", "relations": { "franchise": "film" }
                }
            }
        }
    }
}

wget http://media.sundog-soft.com/es6/series.json

curl -H 'Content-Type: application/json' -X PUT http://{{host}}:9200/_bulk --data-binary @series.json
'

# Query by parent
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/series/movie/_search?pretty -d '
{
    "query": {
        "has_parent": {
            "parent_type": "franchise",
            "query": {
                "match": { "title": "Star War" }
            }
        }
    }
}
'

# Query by child
curl -H 'Content-Type: application/json' -X GET http://{{host}}:9200/series/movie/_search?pretty -d '
{
    "query": {
        "has_child": {
            "type": "film",
            "query": {
                "match": { "title": "The Force Awakens" }
            }
        }
    }
}
```
