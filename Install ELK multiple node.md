#Install ELK multiple node.md

Mục đích: Tách riêng biệt các thành phần trong ELK để dễ dàng quản lý hoặc nâng cấp hệ thống.

### Mô hình triển khai

3 node: 
- **node 1** : Logstash + JAVA
- **node 2**: Elastisearch + JAVA
- **node 3**: Kibana

Cài đặt các mỗi thành phần trên mỗi node theo bài viết [này](https://github.com/huytm/ELK---STACK/blob/master/Cai%20dat%20Logstash%20-%20elasticsear%20-%20kibana.md)

- node 1: **node 1** cấu hình output trỏ ip sang **remote elasticsearch - node 2**

<img src="http://i.imgur.com/vuOB7SW.png">

- node 3: **node 3** sử dụng **remote elasticsearch - node 2**, cấu hình tại file `/var/www/kibana/config.js`

<img src="http://i.imgur.com/UE67an2.png">

- Restast Logstash ở node 1 và Apahce2 ở node 2
