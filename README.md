# elastic-cheatsheet
a cheat sheet for elastic

## Searching
### OR
Find all items that are either 'red' or 'round'. 
```[source,js]
GET index_name/_search
{
    "query": {
        "bool": {
            "should": [
                {"term": {"color": "red"}},
                {"term": {"form": "round"}}
            ]
        }
    }
}
```
### by field existence 
Find all items with a field 'A'.
```[source,js]
GET index_name/_search
{
    "query": {
        "bool": {
            "must": {"exists": {"field": "A"}}
        }
    }
}
```
### by arbitrary field value
Find all items that have some value in field 'A'. (Using the keyword suffix is not mandatory)
```[source,js]
GET index_name/_search
{
    "query": {
        "bool": {
            "must": {"exists": {"field": "A"}},
            "must_not": {"term": {"A.keyword": ""}}
        }
    }
}
```
## Deleting by query
[Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)
### by field value
Delete all documents with value 'unwanted_value' in field 'field'.
```[source,js]
POST index_name/_delete_by_query
{
    "query": {
        "term": {
          "field": {
             "value": "unwanted_value"
           }
        }
    }
}
```
### by time range
Delete all documents for which the date field stores a value greater or equal to the given date.
```[source,js]
POST index_name/_delete_by_query
{
    "query": {
        "range": {
          "date": {
             "gte": "2017-03-01 00:00:00"
           }
        }
    }
}
```
## Reindexing
[Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)

Copy all documents from 'source_index' to 'target_index'
```[source,js]
POST _reindex
{
  "source": {
    "index": "source_index"
  },
  "dest": {
    "index": "target_index"
  }
}
```
