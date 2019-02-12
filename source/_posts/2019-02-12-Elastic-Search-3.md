---
title: Elastic Search - Getting Start 3
date: 2019-02-12 13:41:26
tags: [architecture, elastic]
---

# Search API

- sending search parameters through the _REST request URI_
- sending search parameters through the _REST request body_

<!--more-->

## Request UI

The REST API for search is accessible from the \_search endpoint. This example returns all documents in the bank index:

```bash
curl "http://{{host}}:9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

- \_search : search in the bank index
- q=\* : to match all documents in the index
- sort=account_number:asc : sort the results using the account_number field of each document in an ascending order
- pretty : return pretty-printed JSON results

## Request Body

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```

# Query language

Search all

size defaults to 10

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "size": 1
}
'
```

from is 0-based

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
```

Sort

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
```

# Populate Search

Return two fields, account_number and balance (inside of \_source)

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'
```

## Match Query

Returns the account numbered 20

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": { "match": { "account_number": 20 } }
}
'
```

Returns all accounts containing the term "mill"

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": { "match": { "address": "mill" } }
}
'
```

Returns all accounts containing the term "mill" or "lane" in the address

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": { "match": { "address": "mill lane" } }
}
'
```

A variant of match (match_phrase)
Returns all accounts containing the phrase "mill lane" in the address

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": { "match_phrase": { "address": "mill lane" } }
}
'
```

## Bool query

The bool query allows us to compose smaller queries into bigger queries using boolean logic.

the bool _must_ clause specifies all the queries that must be true for a document to be considered a match

````bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'

composes two match queries and returns all accounts containing "mill" or "lane" in the address
```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
````

Composes two match queries and returns all accounts that contain neither "mill" nor "lane" in the address

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
   "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

returns all accounts of anybody who is 40 years old but doesn’t live in ID(aho)

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool": {
        "must": [
            { "match": { "age": "40" } }
        ],
        "must_not": [
            { "match": { "state": "ID" } }
        ]
        }
    }
}
'
```

# Filter

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool": {
            "must": { "match_all": {} },
            "filter": {
                "range": {
                    "balance": {
                        "gte": 20000,
                        "lte": 30000
                    }
                }
            }
        }
    }
}
'
```

# Aggregations

We set size=0 to not show search hits because we only want to see the aggregation results in the response.

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "aggs": {
        "group_by_state": {
            "terms": {
                "field": "state.keyword"
            }
        }
    }
}
'
```

calculates the average account balance by state
only for the top 10 states sorted by count in descending order
sort on the average balance in descending order

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "aggs": {
        "group_by_state": {
            "terms": {
                "field": "state.keyword",
                "order": {
                    "average_balance": "desc"
                }
            },
            "aggs": {
                "average_balance": {
                    "avg": {
                        "field": "balance"
                    }
                }
            }
        }
    }
}
'
```

group by age brackets (ages 20-29, 30-39, and 40-49), then by gender, and then finally get the average account balance, per age bracket, per gender

```bash
curl -X GET "http://{{host}}:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
    "size": 0,
    "aggs": {
        "group_by_age": {
            "range": {
                "field": "age",
                "ranges": [
                {
                    "from": 20,
                    "to": 30
                },
                {
                    "from": 30,
                    "to": 40
                },
                {
                    "from": 40,
                    "to": 50
                }
                ]
            },
            "aggs": {
                "group_by_gender": {
                    "terms": {
                        "field": "gender.keyword"
                    },
                    "aggs": {
                        "average_balance": {
                            "avg": {
                                "field": "balance"
                            }
                        }
                    }
                }
            }
        }
    }
}
'
```