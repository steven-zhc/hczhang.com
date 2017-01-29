---
title: GORM Query
date: 2017-01-10 20:16:28
tags: [grails,groovy,gorm]
---

## Simple Query
```groovy
def users = User.where {
  password == "testing" || child.password == "fake"
}.list(sort: "loginId", order: "desc")
```

## Operations could be used in query

* !=, <, >, <=, >=
* ==~, =~ (case insensitive)
* !, &&, ||

## Seperate lines are implicitly **And** together
```groovy
def users = User.where {
  loginId =~"%${loginIdPart}%"
  if (fromDate) {
    dateCreated >= fromDate
    }
  }.list()
}
```

## List() supports Named Arguments
* max
* offset
* sort
* order
* ignoreCase
* fetch

```groovy
def users = User.list(sort: 'loginId',
            order: 'desc',
            max: 5,
            offset: 10,
            fetch: [posts: 'eager'])
```

## Magic find and count

findBy() and findAllBy()
```groovy
def users = User.findAllByLoginIdIlike("%${loginId}%")
```

count()
```groovy
def poorPasswordCount = User.countByPassword("password")
```

## Criteria Query
### and() and or()
```groovy
def fetchUsers(String loginIdPart, Date fromDate = null) {
  def users = User.withCriteria {
    and {
      ilike 'loginId', "%${loginIdPart}%"
      if (fromDate) {
        ge "dateCreated", fromDate
      }
  }
}
```

### TODO: insert Query criteria mappings table

### Dynamic queries with criteria
```groovy
def advResults() {
  def profileProps = Profile.metaClass.properties*.name
  def profiles = Profile.withCriteria {
    "${params.queryType}" {
      params.each { field, value ->
        if (profileProps.contains(field) && value) {
          ilike field, "%${value}%"
        }
      }
    }
  }
  return [profiles: profiles]
}
```

## Report-style query projections

```groovy
def tagList = Post.withCriteria {
  createAlias "tags", "t"
  user { eq "loginId", "phil" }
  
  projections {
    groupProperty "t.name"
    count "t.id"
  }
}
```

## Using HQL directly
* find()
* findAll()
* executeQuery()
* executeUpdate()

```groovy
User.findAll("from User u where u.userId = ?", ["joe"] )
```