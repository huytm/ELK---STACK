
##GEOIP

GEOIP = IP Geolocation, là quá trình sử dụng để xác định vị trí địa lý của một IP cụ thể, phục vụ cho nhiều mục đích khác nhau. Nó giúp các nhà quản trị biết được lượng truy cập đến từ đâu, xác định được đối tượng và mức độ quan tâm đến ứng dụng...

Logstash sử dụng GeoIP database để chuyển đổi địa chỉ IP thành một vị trí địa lý gần đúng của địa chỉ IP trên bản đồ. Các dữ liệu được này sau đó được lưu trữ trong Elastich search và Kibana sử dụng nó để hiển thị lên bản đồ. 

##Các bước thực hiện:

Do server của mình có một địa chỉ IP public ra internet nên nó thường xuyên rơi vào tình trạng bị các IP từ nước ngoài dò quét mật khẩu ssh, mình muốn dựa vào thôn tin các file ssh log (/var/log/auth.log) nhận được để xác định được các địa điểm tấn công vào server của mình thì thực hiện như sau.

- Download Geolite database mới nhất và lưu nó vào:

```sh
apt-get install unzip -y
cd /etc/logstash
sudo curl -O "http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz"
sudo gunzip GeoLiteCity.dat.gz
```

- Cấu hình logstash sử dụng GEOIP (Chú ý rằng các filter file phải được **grok filter** theo bài viết [này](https://github.com/huytm/ELK---STACK/blob/master/Grok-filter.md))

`sudo vi /etc/logstash/conf.d/13-ssh.conf`

```sh
filter {
  if [type] == "ssh" {
    grok {
       match => { "message" => "%{SSHFAILED}" }
       match => { "message" => "%{SSHACC}" }
    }
    geoip {
      source => "src_ip"
      target => "geoip"
      database => "/etc/logstash/GeoLiteCity.dat"
      add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
      add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
    }
    mutate {
      convert => [ "[geoip][coordinates]", "float"]
    }
  }
}
```

- Save và restart lại logstash

`sudo service logstash restart`


**Chú ý**: Chú ý trường **`src_ip`** vì nó sẽ phục vụ cho việc hiển thị lên biểu đồ

`SSHFAILED Failed %{WORD:auth_method} for %{USER:username} from %{IP:src_ip} port %{INT:src_port} ssh2`

<img src="http://i.imgur.com/JUVoi9E.png">

---

## Hiển thị lên bản đồ:

- Đối với Kibana 3

<img src="http://i.imgur.com/uoHvHdh.png">


- Đối với Kibana 4

Truy cập vào **Kibana Dashboard** => **Visualize** => *Tile map** => **From a new search**

<img src="http://i.imgur.com/9PF9uwk.png">
