# elasticsearch

## 文档

文档指的是最顶层或者根对象，这个根对象被序列化成`JSON`并存储到`elasticsearch`中，并指定了唯一`ID`

## 文档元数据

* `_index`，文档在哪存放
* `_type`，文档表示的对象类别
* `_id`，文档唯一标识

一个文档的`_index`，`_type`和`_id`唯一标识一个文档

每个文档都有一个版本号，当每次对文档进行修改时(包括删除)，`_version`的值会递增

`_source`，这个字段包含我们索引数据时发送给`elasticsearch`的原始`json`文档

## 索引

索引(动词)就是存储一个文档到一个索引(名词)的操作，把文档存储到索引中以便被检索和查询，新文档会覆盖旧文件

索引是指向一个或多个分片的逻辑命名空间，一个分片为一个底层工作单位(一个`Lucene`实例)

应用程序直接与索引交互而不是与分片交互

索引内的任意一个文档都归属于一个主分片，一个副本分片只是主分片的拷贝(冗余备份)

建立索引时就确定主分片数，副本分片数可以随时修改

存储有同一个文档的主分片和副本分片不会分配到同一个节点中

每个字段的所有数据都是默认被索引的，即每个字段都有为了快速检索设置的专用倒排索引

## 索引文档

```bash
curl -X PUT "localhost:9200/megacorp/employee/1?pretty" -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'
```

## 检索文档

```bash
# megacorp 索引名称
# employee 类型名称
# 1 ID

curl -X GET "localhost:9200/megacorp/employee/1?pretty"

curl -X GET "localhost:9200/megacorp/employee/_search?pretty"

curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith&pretty"

curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'
```

## 水平扩容

在运行的集群中动态调整副本分片数目

```bash
curl -X PUT "localhost:9200/blogs/_settings?pretty" -H 'Content-Type: application/json' -d'
{
    "number_of_replicas" : 2
}
'
```

## 分析器

* 字符过滤器
* 分词器
* `token`过滤器

## ELK

```bash
# elasticsearch docker image
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d --name elasticsearch elasticsearch:7.12.0
# kibana docker image
docker run --link elasticsearch:elasticsearch -p 5601:5601 -d --name kibana kibana:7.12.0
# logstash docker image
docker run --link elasticsearch:elasticsearch -v "$(pwd)\elk\first-pipeline.conf:/usr/share/logstash/common/first-pipeline.conf" --name logstash logstash:7.12.0 logstash -f ./common/first-pipeline.conf --config.reload.automatic
# filebeat docker image
docker run --rm --link logstash:logstash --link kibana:kibana --volume "$(pwd)\elk\filebeat.yml:/usr/share/filebeat/filebeat.yml" --volume "$(pwd)\elk\logstash-tutorial-dataset:/logstash-tutorial.log" --name filebeat elastic/filebeat:7.12.0 filebeat -e -d "publish"
```
