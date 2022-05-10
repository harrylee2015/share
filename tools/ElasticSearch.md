# ElasticSearch

## docker中启动服务

```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node"  -d  elasticsearch:7.12.0
```
## 修改跨域配置

```
docker exec -it elasticsearch /bin/bash
cd /usr/share/elasticsearch/config/
vi elasticsearch.yml
```
在配置文件最后添加
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## ElasticSearch使用方式

* server:9200/index/type/id

* index --> database

* type --> table

通过REST API接口进行GET,POST,PUT,DELETE(获取，添加，修改，删除)
