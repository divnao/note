# ElasticSearch问题思考
## 1. 流处理过程中的Exactly-Once
一种实现机制是：<br />
利用Spark的Checkpoint机制来保证: at least once; <br />
利用HDFS文件覆盖或者ES相同_id的doc会被覆盖来保证: at most once.

## 2. 遇到问题:
### 18.1 ElasticSearch在已有字段mapping的情况下如何动态新增属性？
** 目前已知: 想要支持动态字段扩展, 需要将新字段的`index`设置为`not_analyzed`，并使用动态模板，如下：**
```json
{
    "mappings": {
        "es_type_name": {
            "dynamic_templates": [
                {
                    "template_1": {
                        "match": "*log*",
                        "match_mapping_type": "string",
                        "mapping": {
                            "type": "string"
                        }
                    }
                },
                {
                    "template_2": {
                        "match": "*",
                        "match_mapping_type": "string",
                        "mapping": {
                            "type": "string",
                            "index": "not_analyzed"
                        }
                    }
                }
            ]
        }
    }
}
```
