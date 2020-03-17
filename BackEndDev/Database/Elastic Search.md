# Elastic Search

## 文档

type名约定用_doc

### Create

```json
POST test/_doc //自动生成id
```

```json
PUT test/_doc/1?op_type=create //指定id，如果id存在，会报错
```

```json
POST test/_create/1  //若存在，则失败
```

### Read

```json
GET test/_doc/1
```

```json
GET test/_search 查询全部
```

### Index

```json
PUT test/_doc/1 //文档存在，版本号加1,删除再写入;id不存在，创建新的文档
```

### Update

```json
POST test/_update/1 //原文档上更新，指定字段修改或追加，版本号加1
```

### Delete

```json
DELETE test/_doc/1
```

### Bulk 

```json
POST _bulk
```

单条操作失败并不影响其他操作

### Mget

```json
GET /_mget
{
		"docs": [
			{
				"_index": "test",
				"_id": "1"
			},
			{
				"_index": "test",
				"_id": "2"
			}
		]
}
```

## 分析

### Analyzer API

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "you Know, for Search"
}
```

```json
POST test/_analyze
{
	"field": "title",
	"text": "you Know, for Search"
}
```

```java
POST /_analyze
{
	"tokenizer": "standard",
  "filter": ["lowercase"],
	"text": "you Know, for Search"
}
```

### standard 

- 默认分词器
- 按词切分
- 小写处理

### simple

- 非字母去除
- 小写处理
- 分割线单词被切为两个

### whitespace

- 空格切分
- 大小写保留

### stop 

- 去掉停用词
- 其他同Simple

### keyword

- 不分词

### pattern

- 正则表达式分词
- 小写处理

### english

- 去时态、复数

## 搜索

### 基于term的查询

- 对输入不分词，利用相关度算分公式为每个包含该词项的文档进行相关度算分
- 可以通过Constant Score将查询转化为一个Filter避免算分

```json
POST test/_search
{
  "query":{
    "term":{
      "field1":{
        "value": "..."
      }
    }
  }
}
```

```json
POST test/_search
{
  "query":{
    "term":{
      "field1.keyword":{
        "value": "..."
      }
    }
  }
}
```

```json
POST test/_search
{
	"explain": true,
  "query":{
    "constant_score":{
      "filter":{
        "term":{
          "field1.keyword":"..."
        }
      }
    }
  }
}
```

### 基于全文的查询

```json
POST test/_search
{
  "query":{
    "match":{
      "title":{
        "query": "Matrix reloaded",
        "operator": "AND"
      }
    }
  }
}
```

### 分页查询

```json
GET test/_search
{
	"query":{
		"match_all":{}
	},
	"from":0,
	"size":2
}
```

### 带关键字条件查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
}
```

### 带排序查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
	"sort": [
		{"age":{"order":"desc"}}
	]
}
```

### 带filter查询

```json
GET test/_search
{
	"query":{
		"bool":{
			"filter":[
				{"term":{"age":30}}, //match:分词
        {"range":{"release_date":{"lte":"2019/11/15"}}}
			]
		}
	}
}
```

### 带聚合查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
	"sort": [
		{"age":{"order":"desc"}}
	],
	"aggs":{
		"group_by_age":{
			"terms":{
				"field":"age"
			}
		}
	}
}
```

### 布尔查询

```Json
GET test/_search
{
	"query":{
		"match":{
      "title":{
        "query": "...",
        "operator": "and"
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"match":{
      "title":{
        "query": "...",
        "operator": "or",
        "minimum_should_match": 2
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"bool":{
      "must":[
      	{"match":{"title":""}},
      	{"match":{"overvie":""}}
      ] //shoud, must not。should中为true越多，得分越高
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"bool":{
      "should":[
      	{"match":{"title":""}},
      ],
      "filter":[] //should与filter妙用，用于返回并打分，不满足should，则分数为0
    }
  }
}
```

### 短语查询

```json
GET test/_search
{
	"query":{
		"match_phrase":{"title":"steven zissou"}
		}
}
```

### 多字段查询

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
      "operator": "or"
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title^10","overview^0.1"],
      "tie_breaker": 0.3 
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
			"type":"most_fields"  //sum of, best_fields(default): multiply
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
			"type":"cross_fields" 
		}	
	}
}
```

### QueryString查询

```json
GET test/_search
{
	"query":{
		"query_string":{
			"query": "steve AND jobs",
			"fields": ["title"],
		}	
	}
}
```

### 自定义打分

```json
GET test/_search
{
	"query":{
		"function_score":{
			"query": {
				"multi_match":{
					"query": "steve job",
					"fields": ["title","overview"],
					"operator": "or",
					"type":"most_fields"  
				}
			},
			"functions": [
				{
					"field_value_factor": {
						"field": "popularity",
						"modifier": "log2p",
						"factor": 5
					}
				}
			],
			"score_mode": "sum", //默认multiply,不同field value得分相加
      "boost_mode": "sum"  //与old value相加
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"function_score":{
			"query": {
				"bool":{
					"must" 
				}
			},
			"functions": [
				{
					"gauss":{
						"location":{
							"origin":"31.23916171,121.48789949",
							"scale":"100km",
							"offset":"0km",
							"decay":0.5
						}
					}
				},
				{
					"field_value_factor": {
						"field": "remark_score",
					},
					"weight":0.2
				}
			],
			"score_mode": "sum", //默认multiply,不同field value得分相加
      "boost_mode": "sum"  //与old value相加
		}	
	}
}
```

## Mapping

### 查看mapping

```
GET test/_mappings
```

### 更改Mapping字段

- 新增加字段
  - Dynamic为true，一旦有新增字段的文档写入，Mapping同时被更新
  - Dynamic为false，Mapping不会更新，新增字段数据无法被索引，但数据会出现在_source中
  - Dynamic设为strict，文档写入失败
- 修改已有字段
  - 不允许修改，必须重建索引

|             | true | false | strict |
| ----------- | ---- | ----- | ------ |
| 文档可索引  | Y    | Y     | N      |
| 字段可索引  | Y    | N     | N      |
| Mapping更新 | Y    | N     | N      |

设置

```json
PUT test
{
  "mappings":{
    "_doc":{
      "dynamic":"false"
    }
  }
}
```

### 显式Mapping

```json
PUT test
{
  "mappings":{
    "properties":{
      "field1":{
        "type": "..."
      },
      "field2":{
        "type": "...",
        "index": false //不可被搜索
      },
      "mobile":{
        "type": "keyword",
        "null_value": "NULL" //只有Keyword类型支持设定null_value
      },
      "firstName":{
        "type": "text",
        "copy_to": "fullname" //可以搜索fullname,目标字段不出现在_source中，
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"match":{
			"mobile": "NULL"
		}
	}
}
```

### 类型

- 数组类型

没有数组type，但可以以[ ]插入多个相同类型数据

- 日期类型

```
{
	"type":"date",
	"format":"8yyyy/MM/dd||yyyy/M/dd||yyyy/MM/d||yyyy/M/d"
}
```

- 对象类型

```
{
	"type":"object",
	"properties":{}
}
```

## 过滤与切分

去除html标签

```json
POST _analyze
{
	"tokenizer":"keyword",
  "char_filter":["html_strip"],
  "text": "<b>hello</b>"
}
```

替换

```json
POST _analyze
{
	"tokenizer":"standard",
  "char_filter":[
  	{
  		"type": "mapping",
  		"mappings": ["- => _"]  //中划线替换为下划线
  	}
  ],
  "text": "123-456, I-test!"
}
```

文件路径切分 

```json
POST _analyze
{
	"tokenizer":"path_hierarchy",
  "text": "/root/user/desktop"
}
```

空格切分

```json
POST _analyze
{
	"tokenizer": "whitespace",
	"filter": ["stop","lowercase"],
  "text": ["The rain is a good rain"]
}
```

自定义

```json
PUT test
{
	"setting":{
		"analysis":{
			"analyzer":{
				"my_analyzer":{
					"type": "custom",
					"char_filter": [
						"emoticons"
					],
					"tokenizer": "punctuation",
					"filter": [
						"lowercase",
						"english_stop"
					]
				}
			},
			"tokenizer":{
				"punctuation":{
					"type":"pattern",
					"pattern": "[.,!?]"
				}
			},
			"char_filter":{
				"emoticons":{
					"type":"mapping",
					"mappings":[
						":)=>_happy_",
						":(=>_sad_"
					]
				}
			},
			"filter":{
				"english_stop":{
					"type": "stop",
					"stopwords": "_english_"
				}
			}
		}
	}
}
```

## 相关性

### 自定义词典

```xml
<entry key="ext_dict">http://yoursite.com/getCustomDict</entry> 
```

http请求返回两个头部，一个Last-Modified，一个ETag，一个发送变化，词库就会被更新。

### 同义词

config/analysis-ik放入synonyms.txt

```json
PUT /shop?include_type_name=false
{
   "settings" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 1,
    "analysis": {
      "filter": {
        "my_synonym_filter":{
          "type":"synonym",
          "synonyms_path":"analysis-ik/synonyms.txt"
        }
      },
      "analyzer": {
        "ik_syno":{
          "type":"custom",
          "tokenizer":"ik_smart",
          "filter":["my_synonym_filter"]
        },
        "ik_syno_max":{
          "type":"custom",
          "tokenizer":"ik_max_word",
          "filter":["my_synonym_filter"]
        }
    }  }
   },
   "mappings": {
     "properties": {
       "id":{"type":"integer"},
       "name":{
        "type":"text",
        "analyzer": "ik_syno_max",
        "search_analyzer": "ik_syno" 
       },
       "tags":{"type":"text","analyzer": "whitespace","fielddata":true},
       "location":{"type":"geo_point"},
       "remark_score":{"type":"double"},
       "price_per_man":{"type":"integer"},
       "category_id":{"type":"integer"},
       "category_name":{"type":"keyword"},
       "seller_id":{"type":"integer"},
       "seller_remark_score":{"type":"double"},
       "seller_disabled_flag":{"type":"integer"}
     }
   }
}
```

