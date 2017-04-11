# elastic-cheatsheet
a cheat sheet for elastic

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
