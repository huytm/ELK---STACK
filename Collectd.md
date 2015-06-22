# Về Collectd

Ngoài các bản tin log nhận được, Logstash cũng có thể kết hợp thêm với một thành phần khác để thu thập "metric" các phần cứng hệ thống là **Collectd**. Có thể hiểu như sau, ví dụ tại mỗi thời điểm thông số Ram, cpu, ổ cứng, băng thông mạng của bạn là khác nhau. Metric của các phần cứng là một cặp giá trị: time-value, các thông tin về metric sẽ được **Collectd** thu thập và ghi lại thành các bản tin đẩy lên Logstash server.

https://dl.dropboxusercontent.com/u/6762565/wp/elkoverview.jpg


# Cài đặt 

#### Client - máy cần monitor

```sh
sudo apt-get update
sudo apt-get install collectd collectd-utils
```

Cấu hình collectd

`vi /etc/collectd/collectd.conf`

```sh
Hostname "kvm-hpdl36001"  #hostname máy client
FQDNLookup false
LoadPlugin syslog
<Plugin syslog>
        LogLevel info
</Plugin>
LoadPlugin aggregation
LoadPlugin cpu
LoadPlugin df
LoadPlugin disk
LoadPlugin interface
LoadPlugin memory
LoadPlugin network
LoadPlugin processes
LoadPlugin swap
LoadPlugin users
<Plugin "aggregation">   
  <Aggregation>
    Plugin "cpu"
    Type "cpu"

    GroupBy "Host"
    GroupBy "TypeInstance"

    CalculateSum true
    CalculateAverage true
  </Aggregation>
</Plugin>
<Plugin df>
        Device "/dev/mapper/u--del720--vg-root"  #chú ý dùng câu lệnh df -h
        MountPoint "/"
        FSType "ext4"
        ReportReserved "true"
</Plugin>
<Plugin interface>
        Interface "br1"
        Interface "br2"
        Interface "virbr1"
        #Interface "br1"
        IgnoreSelected false
</Plugin>
<Plugin network>
        Server "IP_ELK_SERVER" "25826"  #Server ELK server
</Plugin>
<Plugin rrdtool>
        DataDir "/var/lib/collectd/rrd"
</Plugin>
<Include "/etc/collectd/collectd.conf.d">
        Filter "*.conf"
</Include>
```

`service collectd restart`


**Chú ý** Collectd cung cấp rất nhiều plugin để monitor các thành phần hệ thống, tuy nhiên có một plugin quan tọng nhất là network, nhở mở và trỏ đúng ip về ELK server


#### ELK server

Tạo Input là collectd

`vi /etc/logstash/conf.d/02-collectd.conf`

- Đối với ELK sử dụng kibana 3

```sh
input {
  collectd {}
}
```

- Đối với ELK sử dụng kibana 4

```sh

input {
  udp {
    port => 25826         
    buffer_size => 1452   
    codec => collectd { } 
    type => "collectd"
  }
}

```

`service logstash restart`

### Sử dụng trên kibana

####1. CPU

- Hệ thống mình sử dụng, hiện đang monitor 3 máy chủ, ở đây mình xin phép ví dụ hiển thị thông tin về cpu của 1 máy chủ dell-720 có hostname là u-del720.
- Plugin sử dụng để thu thập metric cho CPU có nhiều "nhân" là **aggregation**, sử dụng các câu truy vấn thích hợp trên dashboard và vẽ biểu đồ như sau:

Các câu truy vấn sử dụng sẽ là:

```sh
"aggregation" AND host: "u-del720"
"aggregation" AND host: "u-del720" AND type_instance : "idle"
"aggregation" AND host: "u-del720" AND type_instance : "user"
"aggregation" AND host: "u-del720" AND type_instance : "wait"
```

<img src="http://i.imgur.com/3tBp691.png">

<img src="http://i.imgur.com/ptCgrbs.png">

<img src="http://i.imgur.com/sVpksJE.png">


####2. DF - thông số ổ cứng

Như đã nói ở trên, plugin sử dụng để lấy được các thông tin này là df, chỉ có một chú ý khi cấu hình collectd mình đã lưu ý ở trên đó là phải sử dụng câu lệnh df -h để lấy đúng tên ổ cứng sử dụng

<img src="http://i.imgur.com/6WsZVLJ.png">

Các câu truy vấn mình sử dụng là 

```sh
plugin: "df" AND plugin_instance: "root" AND host : "u-del720"
plugin: "df" AND plugin_instance: "root" AND host : "u-del720" AND type_instance: "free"
plugin: "df" AND plugin_instance: "root" AND host : "u-del720" AND type_instance: "used"
plugin: "df" AND plugin_instance: "root" AND host : "u-del720" AND type_instance: "reserved"
```

<img src="http://i.imgur.com/2LgRxbv.png">

<img src="http://i.imgur.com/l10y1oM.png">

<img src="http://i.imgur.com/ag9l6jp.png">


####3. Interface

- Plugin sử dụng interface
- Các câu truy vấn:
- 2 thông tin quan tâm chính là **rx - reciver, tx - transmitter**

plugin: "interface" AND plugin_instance: "br1" AND host: "u-del720"   (Đối với card mạng là br1)
plugin: "interface" AND plugin_instance: "br2" AND host: "u-del720"	  (Đối với card mạng là br2)


<img src="http://i.imgur.com/wgjvfSn.png">

<img src="http://i.imgur.com/4FHmhsy.png">

<img src="http://i.imgur.com/XcEehUd.png">

####4. IOPS - Tốc độ đọc ghi ổ cứng

- Plugin sử dụng: Disk
- Các câu truy vấn:

```sh

<img src="http://i.imgur.com/EgRFFh5.png">

<img src="http://i.imgur.com/zN6xmVF.png">

<img src="http://i.imgur.com/EgRFFh5.png">


####5. RAM

- Plugin sử dụng: Memory
- Các câu truy vấn:


```sh
plugin: "memory" AND type_instance: "free" AND host: "u-del720"
plugin: "memory" AND type_instance: "buffered" AND host: "u-del720"
plugin: "memory" AND type_instance: "cached" AND host: "u-del720"
plugin: "memory" AND type_instance: "used" AND host: "u-del720"
```

<img src="http://i.imgur.com/kF6IBzD.png">

<img src="http://i.imgur.com/ajMWTJQ.png">


