# Collectd-ELK-stack

#Sử dụng collectd kết hợp với  Elk để thu thập các thông tin về performance hệ thống 

Mô hình lab gồm 1 node duy nhất: sử dụng collectd và ELK để monitor chính nó

Mô hình logic các thành phần hoạt động như sau:

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/elkoverview.jpg">

###A. Cài Đặt

- Cài đặt ELK tại [đây](https://github.com/huytm/Cai-dat-ELK) 

- Tại trường input của file `vi /etc/logstash/conf.d/10-syslog.conf` thêm nội dung sau:

`collectd {}`

- Cài đặt collectd:

```sh
sudo apt-get update
sudo apt-get install collectd collectd-utils
```

- Cấu hình collectd

`sudo vi /etc/collectd/collectd.conf`

```sh
Hostname "client-hostname"
FQDNLookup false

#Enable các plugin cần thu thập thông tin
LoadPlugin apache
LoadPlugin cpu
LoadPlugin df
LoadPlugin entropy
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin processes
LoadPlugin rrdtool
LoadPlugin users
LoadPlugin network

<Plugin network>
Server "IP_ELK_server" "25826"
</Plugin>

<Plugin interface>
       Interface "eth0 (Nic cần monitor)"
       IgnoreSelected false
</Plugin>

<Plugin df>
        Device "/dev/mapper/Logstash--server--vg-root" #sử dụng lệnh df -h để lấy đúng thông tin
        MountPoint "/"
        FSType "ext4"
        ReportReserved "true"
</Plugin>
```

- Khởi động lại collectd 

`service collectd restart`


###B Sử dụng dashboard
Với mỗi plugin được thu thập chúng ta sử dụng từng queries sau để vẽ biểu đồ tương ứng

####CPU
- plugin: cpu
- type_instance: wait, system, softirq, user, interrupt, steal, idle, nice
- plugin_instance: 0, 1, 2, 3

**Kibana queries**

```sh
plugin: "cpu" AND plugin_instance: "0"
plugin: "cpu" AND plugin_instance: "1"
plugin: "cpu" AND plugin_instance: "2"
plugin: "cpu" AND plugin_instance: "3"
```

sau đó sử dụng các query này vẽ biểu đồ như sau

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-cpu.jpg">

Thực hiện với từng query ta sẽ được kết quả như sau

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-cpu.jpg">

####df 

- plugin: df
- type_instance: reserved, used, free
- plugin_instance: root

**Kibana queries**

`plugin: "df" AND plugin_instance: "root"`

thực hiện tương tự như vẽ biểu đồ với CPU trên ta có:

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-disk.jpg">

#### Interface

- plugin: interface
- plugin_instance: eth0
- addtnl attributes: rx, tx

Kibana queries

`plugin: "interface" AND plugin_instance: "eth0"`

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-rx.jpg">

làm tương tự với 2 thông số là rx và tx ta được kết quả như sau:

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-interface.jpg">

#### Memory

- plugin: memory
- type_instance: free, buffered, cached, used

Kibana queries

```sh
plugin: "memory" AND type_instance: "free"
plugin: "memory" AND type_instance: "buffered"
plugin: "memory" AND type_instance: "cached"
plugin: "memory" AND type_instance: "used"
```

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-memory.jpg">

Được kết quả như sau:

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-memory.jpg">

#### Swap

- plugin: swap
- type_instance: cached, in, free, out, used

Kibana queries

```sh
plugin: "swap" AND type_instance: "free"
plugin: "swap" AND type_instance: "in"
plugin: "swap" AND type_instance: "out"
plugin: "swap" AND type_instance: "cached"
plugin: "swap" AND type_instance: "used"
```

Thực hiện tương tự với plugin memory ta được kết quả như sau

<img src="https://dl.dropboxusercontent.com/u/6762565/wp/kibana-display-swap.jpg">

Đây là các plugin cần thiết để hiển thị, ngoài ra bạn cũng có thể hiển thị các thông tin cần thiết khác tùy vào hoàn cảnh sử dụng

-----

Tham khảo:

- https://mtalavera.wordpress.com/2015/02/16/monitoring-with-collectd-and-kibana/
