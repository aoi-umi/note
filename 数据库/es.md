# ElasticSearch

## docker 安装

### elasticsearch

```sh
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.3.2

docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2

# 进入容器
# 修改配置文件
vi config/elasticsearch.yml

# 加入跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"

```

### elasticsearch-head

```sh
docker pull mobz/elasticsearch-head:5

docker run -d --name es_admin -p 9100:9100 mobz/elasticsearch-head:5
```

[参考](https://www.jianshu.com/p/7d687c9dba4f)

### 创建映射

```sh
#创建映射
post http://localhost:9200/索引库名称/类型名称/_mapping

curl -XPOST  http://192.168.100.164:9200/test/doc/_mapping -H "Content-Type: application/json" -d '{
"properties": {
    "name": {
        "type": "text"
    },
 }
}'

#插入数据
curl -XPOST  http://192.168.100.164:9200/test/doc -H "Content-Type: application/json" -d '{
    "name":"test",
}'
```
