## ElasticSearch

 docker run -di -p 9200:9200 -p 9300:9300 --name es -e 'discovery.type=single-node' -e 'cluster.name=elasticsearch' -v ~/notehub/elasticsearch/plugins:/usr/share/elasticsearch/plugins -v ~/notehub/elasticsearch/data:/usr/share/elasticsearch/data docker.io/elasticsearch:6.8.4

 docker run -di --name kibana --net esnetwork -e ELASTICSEARCH_URL=http://192.168.0.127:9200 -p 5601:5601 docker.io/kibana:6.8.4

### Kibaba

### 加密

elasticsearch.yml

```yml
xpack.security.enabled: true
```

在bin目录下输入

```shell
sh elasticsearch-setup-passwords interactive
```

配置kibana.yml

```yml
elasticsearch.username: "elastic"
elasticsearch.password: "969023014"
```

