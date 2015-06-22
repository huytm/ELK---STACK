#Elasticsearch_cluster

#Elasticsearch_cluster

Một yêu cầu đặt ra đối với bất kỳ hệ thống nào đó là: Đã có hệ thống rồi, vận hành và duy trì nó thế nào? Trong hệ thống ELK triển khai, thành phần quan trọng nhất là Elasticsearch. Elasticsearch vừa nhận nhiệm vụ lưu trữ bản tin đến, vừa phải thực hiện để tìm kiếm, truy vấn. Logstash và Kibana đều hoạt động dựa trên Elasticsearch, do đó Elasticsearch là thành phần tối quan trọng cần phải theo dõi sát sao để đảm bảo hệ thống hoạt động ổn định

Do đã nói ở trên, Elasticsearch rất quan trọng, muốn đảm bảo cho nó hoạt động liên tục và có khả năng chống chịu lỗi gây mất mát dữ liệu, khi triển khai một hệ thống lớn, Elasticsearch nên được cấu hình thành một cụm - cluster. Cách thức hoạt động chi tiết xem tại [đây](http://stackoverflow.com/questions/15694724/shards-and-replicas-in-elasticsearch). Mình có thể tóm tắt theo ý mình hiểu như sau: Elasticsearch khi triển khai, bản thân nó đã được coi là *1 cluster với 1 node duy nhất*, nó tạo ra *5 shards để chứa dữ liệu của bạn*. Lưu ý là chỉ có 5 shards theo mặc định. Tuy nhiên để đảm bảo dữ liệu được an toàn, không bị mất mát, elasticsearch muốn nhân bản - replicas mỗi 1 shards ra một bản sao khác. Khi này 5 shards ban đầu là không đủ, cần phải thêm một node elasticsearch mới để tạo ra 5 shards phục vụ cho việc lưu trữ các bản sao. Từ các lần sau, elasticsearch khi truy vấn sẽ sử dụng replicas lưu trữ ở node 2 hoặc primary shard lưu trữ ở node 1. Điều này làm tăng performance cho hệ thống.

## Cài đặt.

Mô hình cluster của mình trong ví dụ này gồm 2 node.

# Mô hình Ha với elasticsearch cluster


- node 1: Cài đặt JAVA - logstash - Kibana
- [Cluster]: gổm 2 node : node A - JAVA + ELASTICSEARCH, node B -JAVA + ELASTICSEARCH



- Kibana trỏ ip đến IP của một trong 2 node elasticsearh
- Logstash trỏ output `elasticsearch { cluster => "OurCluster" }`

<img src="http://i.imgur.com/FdBeW0x.png">

- node A cấu hình `vi /etc/elasticsearch/elasticsearch.yml` thêm 2 dòng sau

```sh
cluster.name: OurCluster
node.name: "esa"
node.master: true
```
<img src="http://i.imgur.com/MkJcp1a.png">

- node B cấu hình `vi /etc/elasticsearch/elasticsearch.yml` thêm 2 dòng sau

```sh
cluster.name: OurCluster
node.name: "esa"
node.master: false
```

<img src="http://i.imgur.com/lulKHLE.png">

Restart `elasticsearch` trên cả 2 node A và B

Tại server kiểm tra lại bằng câu lệnh:

`curl http://localhost:9200/_nodes/process?pretty` nếu thấy có 3 node (node logstash, node A, node B là OK)

hoặc vào giao diện web `http://localhost:9200/_cluster/health?pretty=true` 

**Chú ý** localhost ở đây là ip của elasticsearch node, ví dụ node A, node B

<img src="http://i.imgur.com/BMTINIW.png">

thấy status = green là OK
