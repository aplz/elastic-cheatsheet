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
### with must and should
Find all items that are round and either 'red' or 'round'. 
```[source,js]
GET index_name/_search
{
    "query": {
        "bool": {
            "must": [
                {"term": {"shape": "round"}},
                {"bool": {
                        "should": [
                            {"term": {"color": "red"}},
                            {"term": {"color": "green"}}
                        ]
                    }
                }
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
### sorted by nested time stamp
Fetch items from index 'index' sorted by the time stamp in the nested field 'time'.
```[source,js]
GET index/_search
{
  "query": {"match_all": {}},
  "sort": [
    {
      "path_to_field.time": {
        "order": "desc",
        "nested_path": "path_to_field"
      }
    }
  ]
}
```
### by time range / time of day
Find all events happening between 8.00am and 10.00am
```[source,js]
GET index_nested/_search
{
  "query": {
        "bool": {
          "must": [
            {
              "script": {
                "script": {
                    "inline" : "doc['date_field'].date.hourOfDay >= params.min && doc['order_date'].date.hourOfDay < params.max",
                    "params": {
                      "min": 8,
                      "max": 10
                     }
                 }
              }
            }
          ]
       }
    }
}
```
### in multiple fields with a regex
Search for a term 'A' in all fields with named something like 'field_xyz' and return only those fields.
```[source,js]
GET index/_search
{
  "_source": ["field_*"], 
  "query": {
    "multi_match" : {
      "query":    "A",
      "fields": [ "field_*" ] 
    }
  }
}
```

## More Like This
[More Like This Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html)
### more control when querying different fields
The MoreLikeThis query automatically selects the best field for a match. This is not detailed in the documentation but has been observed by me and also stated in [this discussion](https://github.com/elastic/elasticsearch/issues/13654). Since it is not obvious _why_ a specific field is chosen, you can gain some control by combining multiple MoreLikeThis queries in a boolean query that you can boost as you like. 
### with nested fields
Assuming indexed documents are baskets with articles as nested fields, and the goal is to find similar baskets to the one specified by 'basket_id' (based upon the article's brand, color and size), a query might look like this.
```[source,js]
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
### by missing value
Delete all documents with that have no value in field 'field'.
```[source,js]
POST index_name/_delete_by_query
{
    "query": {
        "term": {
          "field": {
             "value": ""
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
Note that this will *not* respect any mappings you defined for source_index. The mappings in target_index will be inferred automatically. If you want to preserve the original mapping, you have to create target_index *before* reindexing with this mapping. 
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
## Custom Analyzers
Combine a [GermanAnalyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html#german-analyzer) with a [SynonymFilter](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/analysis-synonym-tokenfilter.html)
```[source,js]
PUT target_index
{
  "settings": {
    "index": {
      "analysis": {
        "filter": {
          "german_stop": {
            "type": "stop",
            "stopwords": "_german_"
          },
          "german_stemmer": {
            "type": "stemmer",
            "language": "light_german"
          },
          "synonym_filter": {
            "type": "synonym",
            "synonyms_path": "synonyms.txt"
          }
        },
        "analyzer": {
          "custom_german": {
            "tokenizer": "standard",
            "filter": [
              "lowercase",
              "german_stop",
              "german_normalization",
              "german_stemmer",
              "synonym_filter"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    "type": {
      "properties": {
        "text": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            },
            "de_custom": {
              "type": "text",
              "analyzer": "custom_german"
            }
          }
        }
      }
    }
  }
}
```
The settings above will have everything in `text` field run through a GermanAnalyzer and SynonymFilter and store the result in the field `text.de_custom`. 
This [medium post](https://medium.com/@lucasmagnum/elasticsearch-setting-up-a-synonyms-search-facea907ef92) provides a more detailed explanation.  
**Note**: The file 'synonyms.txt' **must** be located relative to the CONF_DIR of your elastic instance. See [here](https://discuss.elastic.co/t/500-error-synonym-file-cant-be-read/62345/2) for instance.

