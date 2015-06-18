# Đôi điều về Grok-filter

Ở bài viết[trước](https://github.com/huytm/ELK---STACK/blob/master/README.md) mình đã thực hiện cài đặt và cấu hình *Logstash*, *Elastichsearch*, *Kibana* hoạt động để thực hiện thu thập log. Vậy bản tin log khi được hệ thống nhận được sẽ có nhiều định dạng khác nhau? Người dùng liệu có thể định dạng lại bản tin này cho phù hợp để phục vụ việc phân tích log sau này không? Câu trả lời là được.

Trở lại với **Phương pháp xử lý** log của Logstash **`INPUT => FILTER => OUTPUT`** . Thật may là logstash cung cấp cho người dùng một cơ chế lọc cực mạnh được gọi là **Grok**. Grok hoạt động bằng cách phân tích các bản tin thành các trường riêng biệt và gán cho chúng một định danh. Ví dụ bạn nhận được bản tin là `Openstack is my life`, và bạn muốn chia bản tin này thành nhiều trường `f1 f2 f3 f4` tương ứng thì bạn phải sử dụng **Grok filter** của Logstash.

Một ví dụ cụ thể hơn: Đây là bản tin log của ssh (***/var/log/auth.log***) mà server nhận được:

`Failed password for root from 54.179.13.13 port 39174 ssh2`

Mình chỉ muốn lấy mỗi trường **IP (54.179.13.13)** để đếm số lần IP này cố gắng tấn công dò mật khẩu ssh vào server mình. Do đó mình cần phải "define" lại bản tin đến thành nhiều trường riêng biệt như sau:

```sh
Failed password for uvdc from 172.16.69.1 port 59738 ssh2
=>
Failed %{WORD:auth_method} for %{USER:username} from %{IP:src_ip} port %{INT:src_port} ssh2

```

*Mẹo* Sử dụng trang này để kiểm tra `http://grokdebug.herokuapp.com/`

Kết quả:
Bản tin nhận được lúc đầu:

<img src="http://i.imgur.com/goVH1Am.png">

Bản tin nhận được sau khi được định dạng lại:

<img src="http://i.imgur.com/WfW18Vm.png">

Như đã thấy trên hình, mình đã tách được IP truy cập đến thành 1 trường riêng biệt là `src_ip`, port thành `src_port` ...

Vậy để thực hiện việc này thế nào? Trước hết cũng cần phải hiểu cách thức làm việc của chúng 1 chút

Thứ 1: Mỗi bản tin log từ Client sử dụng logstash-forwarder người dùng có "gắn" cho nó một **type** riêng biệt.

<img src="http://i.imgur.com/FuIOiU1.png">

Thứ 2: Bản tin log mà logstash nhận được từ logstash-forwarder sẽ được Filter, Grok-filter sẽ sử dụng chính các type ở trên để thực hiện lọc bản tin.

Thứ 3: Các mẫu bộ lọc cho từng bản tin được lưu trữ ở đâu? Logstash cung cấp cho người dùng một folder chứa các bộ lọc gọi là `patterns`, người dùng có thể thêm các bộ lọc cho từng bản tin đặc biệt hoặc sử dụng bộ lọc có sẵn (Ví dụ như Apache2 là có sẵn).

---

#Thực hiện

Thực hiện cài đặt bộ sản phẩm tại [bài viết trước]](https://github.com/huytm/ELK---STACK/blob/master/README.md)) của mình.

Ở bài viết này mình sẽ lọc một vài bản tin log cụ thể đó là **Nginx**, **SSH**, **APACHE**

#### Chuẩn bị

Tại **ELK server**, tạo Thư mục `patterns`

```sh
sudo mkdir -p /opt/logstash/patterns
sudo chown logstash:logstash /opt/logstash/patterns
```

#### 1. Với Nginx

#####- Tại **Client**

`sudo vi /etc/logstash-forwarder.conf`

```sh
{
  "network": {
        "servers": [ "172.16.69.210:5000" ],  # "servers": [ "IP_ELK_SERVER:PORT" ]
        "timeout": 15,
        "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt" 
  },

  "files": [
{
      "paths": [
       "var/log/nginx/access.log"
       ],
      "fields": { "type": "nginx-access" }
    }
 ]
}

```

`sudo service logstash-forwarder restart`

##### Tại **ELK-Server**

`sudo vi /opt/logstash/patterns/nginx`

```sh
NGUSERNAME [a-zA-Z\.\@\-\+_%]+
NGUSER %{NGUSERNAME}
NGINXACCESS %{IPORHOST:clientip} %{NGUSER:ident} %{NGUSER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response} (?:%{NUMBER:bytes}|-) (?:"(?:%{URI:referrer}|-)"|%{QS:referrer}) %{QS:agent}

```

`sudo chown logstash:logstash /opt/logstash/patterns/nginx`

chú ý: **NGINXACESS** tức là pattern sẽ được sử dụng để "grok-filer" trong logstash

- Bây giờ tạo file filter cho nginx

`sudo vi /etc/logstash/conf.d/11-nginx.conf`

```sh
filter {
  if [type] == "nginx-access" {
    grok {
      match => { "message" => "%{NGINXACCESS}" }
    }
  }
}

```

`sudo service logstash restart`

***Chú ý*** Tại server: logstash hoạt động như sau `input => filer => output` tương đương với `02-logstashfowarder.conf =>  11-nginx.conf => 30-output.conf`

---

### 2. Với APACHE:

#####- Tại **Client**

Đẩy file apache `access.log` lên server

`sudo vi /etc/logstash-forwarder.conf`

```sh
{
  "network": {
        "servers": [ "172.16.69.210:5000" ],  # "servers": [ "IP_ELK_SERVER:PORT" ]
        "timeout": 15,
        "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt" 
  },

  "files": [
{
      "paths": [
       "var/log/apache2/access.log"
       ],
      "fields": { "type": "apache-access" }
    }
 ]
}

```

`sudo service logstash-forwarder restart`

 ####- Tại **SERVER**

`sudo vi /etc/logstash/conf.d/12-apache.conf`

```sh
filter {
  if [type] == "apache-access" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
}
```

`sudo service logstash restart`

**Chú ý**: Mặc định Apache pattern đã là một mẫu có sẵn trong logstash nên không cần bước tạo pattern như với nginx


---

###3. Với SSH:

 #####- Tại **Client**

`sudo vi /etc/logstash-forwarder.conf`

```sh
{
  "network": {
        "servers": [ "172.16.69.210:5000" ],  # "servers": [ "IP_ELK_SERVER:PORT" ]
        "timeout": 15,
        "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt" 
  },

  "files": [
{
      "paths": [
       "/var/log/auth.log"
       ],
      "fields": { "type": "ssh" }
    }
 ]
}

```

`sudo service logstash-forwarder restart`


 #####- Tại **Server**

`sudo vi /opt/logstash/patterns/ssh`

```sh
SSHFAILED Failed %{WORD:auth_method} for %{USER:username} from %{IP:src_ip} port %{INT:src_port} ssh2

SSHACC Accepted %{WORD:auth_method} for %{USER:username} from %{IP:src_ip} port %{INT:src_port} ssh2

SSHFU Failed %{WORD:auth_method} for invalid user %{USER:username} from %{IP:src_ip} port %{INT:src_port} ssh2
```

`sudo chown logstash:logstash /opt/logstash/patterns/ssh`

- Tạo file filter cho ssh

`sudo vi /etc/logstash/conf.d/13-ssh.conf`

```sh
filter {
  if [type] == "ssh" {
    grok {
      match => { "message" => "%{SSHFAILED}" }
      match => { "message" => "%{SSHACC}" }
      match => { "message" => "%{SSHFU}" }    
    } 
  }
}

```

`sudo service logstash restart`

--- 




