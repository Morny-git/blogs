1./_search
	在所有的索引中搜索所有的类型
2./gb/_search
	在 gb 索引中搜索所有的类型
3.gb,us/_search
	在 gb 和 us 索引中搜索所有的文档
4./g*,u*/_search
	在任何以 g 或者 u 开头的索引中搜索所有的类型
5./gb/user/_search
	在 gb 索引中搜索 user 类型
6./gb,us/user,tweet/_search
	在 gb 和 us 索引中搜索 user 和 tweet 类型
7./_all/user,tweet/_search
	在所有的索引中搜索 user 和 tweet 类型
8.size
	显示应该返回的结果数量，默认是 10
	from
	显示应该跳过的初始结果数量，默认是 0
9.GET /gb/_mapping/tweet
	查看文档结构
10.es的数据分为：精确值  和 全文
11.使用analyze api 查看文本如何被分析。可指定分析器与分析的文本。
	GET /_analyze
	{
	  "analyzer": "standard",
	  "text": "Text to analyze"
	}
	查询gb索引，tweet类型的分词器工作情况
	GET /gb/_analyze
	{
	  "field": "tweet",
	  "text": "Black-cats" 
	}
12.创建一个新索引，指定 tweet 域使用 english 分析器
	PUT /gb 
	{
	  "mappings": {
		"tweet" : {
		  "properties" : {
			"tweet" : {
			  "type" :    "string",
			  "analyzer": "english"
			},
			"date" : {
			  "type" :   "date"
			},
			"name" : {
			  "type" :   "string"
			},
			"user_id" : {
			  "type" :   "long"
			}
		  }
		}
	  }
	}
13.使用 _mapping 在 tweet 映射增加一个新的名为 tag 的 not_analyzed 的文本域
	PUT /gb/_mapping/tweet
	{
	  "properties" : {
		"tag" : {
		  "type" :    "string",
		  "index":    "not_analyzed"
		}
	  }
	}
14.排序：与query平级别.
	"sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }},
		"dates": { "order": "asc",  "mode":  "min" }//多值字段的排序
    ]
15.should中的boost。增加语句的评分权重
	 "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }}
            ]
	
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
15.multi_match 查询可以在多个字段上执行相同的 match 查询
	{
		"multi_match": {
			"query":    "full text search",
			"fields":   [ "title", "body" ]
		}
	}
16.查询特定兴趣爱好员工的平均年龄
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
17.filter过滤器不计算评分
17.1 term过滤  = 
17.2 terms过滤  in
17.3 range范围查询 
gt -大于 gte 大于等于 lt 小于lte 小于等于
17.4 Exists，查询文档包含或者不包含某个字段值，类似 SQL的is null
17.5 missing (5.2.2之后取消)不存在该字段（must_not +exists = missing）
17.6 bool过滤  must   must_not  should 
17.7 match_all查询。查询所有文档，是没有查询条件的默认句
17.8 match查询。match在全文本查询和精确值查询都适用。match查询之前会用分析器先分析一下查询的字符串。 全文检索
17.9 multi_match。在查询的基础上同时搜索多个字段  
17.10  match_phrase  短语搜索   like 
17.11 aggs = group by
17.12 constant_score 执行只有filter的查询。代替bool
17.13
#复杂结构查询

```
{
    "query": {
        "bool": {
            "must_not": [],
            "must": [
                {
                    "term": {
                        "qid": 2898177794
                    }
                },{
                    "term": {
                        "pid": "c9f85d8f528ad1c2a8ce0f3f8f43b3bd"
                    }
                },{
				  "nested": {
					"path": "choiceFields",
					"query": {
					  "bool": {
						"must": [
						  {
							"term": {
							  "choiceFields.fieldName": "imgSizeRate"
							}
						  },{
							"term": {
							  "choiceFields.fieldValue": "0.8"
							}
						  }
						]
					  }
					}
				  }
				}                

    ]
    }
},
//高亮显示
"highlight": {
    "fields" : {
        "about" : {}
    }
}

}
```


