#ElasticHQ

Ở bài trước mình đã nói đến việc cài đặt elasticsearh thành một cụm [cluster](https://github.com/huytm/ELK---STACK/blob/master/Elasticsearch_cluster.md) nhằm tăng khả năng chống chịu lỗi và performance của hệ thống.

ElasticHQ là công cụ để giám sát, quản lý, truy vấn  bằng giao diện web cho elasticsearh, hoặc một cụm elasticsearh. Dựa vào các thông tin hiển thị trên web, có thể quyết định được tình trạng "health" của Cluster elasticsearh.

###Cài đặt

- Tại master node elasticsearh:


```sh
cd /usr/share
sudo elasticsearch/bin/plugin -install royrusso/elasticsearch-HQ
```

- Truy cập vào địa chỉ `ip_node_master:9200/_plugin/HQ/`

<img src="http://i.imgur.com/fmJAQoF.png">


- Ngoài ElasticHQ, còn có plugin khác như Marvel, head...

