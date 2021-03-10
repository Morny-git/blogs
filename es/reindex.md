#### 查询索引字段

GET _mapping 可以查询到索引的字段情况

#### 查询索引配置

GET _settings 

#### 创建索引

PUT /index_1_v1

```
{
		"settings": {
			"number_of_shards": 50,
			"number_of_replicas": 0,
			"refresh_interval": -1,
			"analysis": {
				"analyzer": {
					"optimizeIK": {
						"type": "custom",
						"tokenizer": "ik_max_word"
					}
				}
			}
		},
        "mappings": {
			"doc": {
                "dynamic": "true",
                "properties": {
						"address": {
                        "type": "text",
                        "fields": {
                            "raw": { "type": "keyword"}，
                            "sug": { "type": "completion"}
                        },
                        "analyzer": "optimizeIK"
                    }
					……		
			}
		}
	}
}
```

#### dynamic 

- 动态映射（dynamic：true）：动态添加新的字段（或缺省）。
- 静态映射（dynamic：false）：忽略新的字段。在原有的映射基础上，当有新的字段时，不会主动的添加新的映射关系，只作为查询结果出现在查询中。
- •严格模式（dynamic： strict）：如果遇到新的字段，就抛出异常

#### 添加字段

```
POST index_1/doc/_mapping
	{
		"properties": {
		  "startdate": {
			  "type": "date",
			  "format": "yyyy-MM-dd HH:mm:ss"
		  }
	  }
	}
```



#### 添加字段sss并赋初始值

```
POST	index/_update_by_query 
	{
	 "script": {
	  "lang": "painless",
	  "inline": "ctx._source.sss = params.sss",
	  "params": {
		"sss": 1
			}
		}
	}
```

#### 查看分词情况

POST  _analyze

```
{
"analyzer":"ik_max_word",
"text":"中华人民共和国“
}
```

" ik_max_word " ： "中华人民共和国,中华人民,中华,华人,人民共和国,人民,共和国,共和,国,国歌“

" ik_samrt " ： "中华人民共和国,国歌"

#### reindex

POST  _reindex?slices=50&wait_for_completion=false

slices：最好等于索引的分片数，等于分片数时速度最快

wait_for_completion：设置为false,则会进行预执行检查，执行请求后返回一个taskId.

requests_per_second=500 ： 限制操作频率

```
{
"conflicts": "proceed",//有冲突时忽略，继续进行
 "source": {
 "index": "first_index",
 "type": "first_type",
 "size":10000,//reindex 底层是scroll,默认1000
 "query": {  // query的条件
      "term": {
        "name": "name_value"
      }
    }
 },
 "dest": {
 "index": "second_index",
 "type": "second_type",
  "op_type": "create",//dest中没有的doc创建到dest里
  "version_type":"external"//internal表示source全覆盖，冲突时version+1， external表示谁大用谁
 },
 "script": {//同步时，做数据梳理
 "lang": "painless",
 "inline": "ctx._source.newName = ctx._source.remove(\"name\");ctx._source.status = params.status;ctx._source.audit = params.audit",//将name的重命名为newName.将source中audit的值写入到dest
// 存在则修改。if(ctx._source.containsKey('game_type')){ctx._version++;ctx._source.gametype = ctx._source.remove('game_type')}
 "params": { "sss": "2019-04-09" }
 }
}
```

#### 查看task

(1)GET _tasks?detailed=true&actions=*reindex

(2)GET /_tasks/r1A2WoRbTwKZ516z6NEs5A:36619

#### 查询count

GET/index_1_v1/ _count

#### 设置别名

```
POST /_aliases
{
 "actions" : [
   { "remove" : { "index" : "index_1_v2", "alias" : "index_1" } },
  { "add" : { "index" : "index_1_v1", "alias" : "index_1" } }
 ]
}
```

#### reindex后续

```
PUT /index_4/_settings
{
  "number_of_replicas": 1，
  "refresh_interval": 1
}
```

#### search

- {"exists" : { "field" : “fieldA"}}

- {"term" : { " fieldA " : “xxx"}}

- {"terms": {" fieldA ": [""xx","yy"]}},

- {"range":{" fieldA ":{"gte":"22","lte":"33"}}}// gt 	gte	 lt	 lte

- {"match_phrase_prefix":{"bName":"eee"}}

   match：全文检索

   match_phrase：短语匹配

   match_phrase_prefix:和match_pharse类似,不过支持最后一项使用前缀匹配

```
POST _search
{
  "query": {
      "bool": {
        "must":[
            { "match_phrase_prefix" : { "pName" : "72" }
            }
          ]      
  		}
 	 }
}
```

#### 聚合aggs 类似groupby

```
 "aggs": {
        "my_name": {//返回出的bulk的名字
            "terms": {
                "size": 10000,
                "field": “fieldA",//使用fieldA来分组
                 "order" : {  "_term" : "asc" }
            }
        }
    }

```

#### 自动补全

##### term-suggester

![image-20210226173250970](..\image\es\term-suggester.png)

- Missing:只有词典里找不到词，才会为其提供相似的选项
- Popular:即使词典中存在也提示更高的相似项
- Always：不管token是否存在于索引词典里都要给出相似项

#####  phrase-suggester

Phrase suggester在Term suggester的基础上，会考量多个term之间的关系，比如是否同时出现在索引的原文里，相邻程度，以及词频等等

![image-20210226180107640](D:\project\blog\docs\image\es\phrase-suggester.png)

##### completion-suggester

自动补全，为实现自动搜索补全，在创建index时设立专门的格式。实现上它和前面两个Suggester采用了不同的数据结构，索引并非通过倒排来完成，而是将analyze过的数据编码成FST和索引一起存放。对于一个open状态的索引，FST会被ES整个装载到内存里的，进行前缀查找速度极快。但是FST只能用于前缀查找，这也是Completion Suggester

![image-20210226180239938](D:\project\blog\docs\image\es\completetion-suggester.png)

##### FST

对于Completion类型的suggest来说，使用的是FST（Finite State Transducers）存储结构，即有穷状态转换器，类似于前缀树或者字典树
 我们有如下几个词：mop、moth、pop、star、stop和top 5个词，那么构建的FST如下图

![image-20210226180417725](D:\project\blog\docs\image\es\FSTpng)

#### 深度分页

```
POST  _search?scroll=1m//m-分钟   s-秒
{
 "size":100,//每次查询出的刷零
 "query":{},
 "slice":{
 	"id":0,//从0开始
 	"max":5 //最大为分片的数量
 }
}
```

后续通过scrollId查询出后续的数据

```
POST _search/scroll
{
  "scroll":"1m",
  "scroll_id":""
}
```

全部查询完后清除scroll,释放内存

```
DELETE _search/scroll/XXX
```

#### 通过查询删除

```
POST _delete_by_query?slices=50&conflicts=proceed&wait_for_completion=false&requests_per_second=-1&scroll_size=5000
{
   "query": {
     "bool": {
       "must": [
			...
        ],"must_not": [
			...
        ],"should": [
			...
        ]
     }
    }
}
```

#### PUT、POST  新增时声明外索引   es:versionType   或内索引

**当es中现有version为8**
version=8&version_type=force  数据更新   version =8
version=8&version_type=external_gte  数据更新   version =8
version=8&version_type=external_gt  数据更新   error  [doc][1]: version conflict, current version [8] is higher or equal to the one provided [8]
version=8&version_type=external 数据更新   error  [doc][1]: version conflict, current version [8] is higher or equal to the one provided [8]

version=1&version_type=force  数据更新   version =1  6.2.2 不再使用
version=1&version_type=external 数据更新   error  [doc][1]: version conflict, current version [1] is higher than the one provided [8]
version=1&version_type=external_gt 数据更新   error  [doc][1]: version conflict, current version [1] is higher than the one provided [8]
version=1&version_type=external_gte 数据更新   error  [doc][1]: version conflict, current version [1] is higher than the one provided [8]