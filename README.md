# elastic-cheatsheet
a cheat sheet for elastic

Even more cheats: http://elasticsearch-cheatsheet.jolicode.com/

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
## More Like This
[More Like This Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html)
### with nested fields
Assuming indexed documents are baskets with articles as nested fields, and the goal is to find similar baskets to the one specified by 'basket_id' (based upon the article's brand, color and size), a query might look like this.
```
{
"query": {
    "nested":{
        "path":"articles",
        "query":{
            "more_like_this" : {
                "fields" : ["articles.brand", "articles.color", "articles.size"],
                "like" : [
                {
                    "_index" : "basket_index",
                    "_type" : "basket",
                    "_id" : "basket_id"
                }
                ],
                "min_term_freq" : 1,
                "max_query_terms" : 20
            }
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
Note that this will *not* respect any mappings you defined for source_index. The mappings in target_index will be inferred automatically. If you want to preserve the original mapping, you have to create target_index with this mapping. 
```[source,js]
PUT target_index 
{
  "mappings" : {
    "articles" : {
      "type": "nested",
      "properties" : {
        "id" : { "type" : "string", "index" : "not_analyzed" }
       }
     }
   }
}   
```
