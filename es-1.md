## Cài đặt Elasticsearch trên nhiều nền tảng

*Phiên bản mới nhất thời điểm hiện tại là: 7.10.2*

### Ubuntu

Bước 1:\
Cập nhật hệ thống
> sudo apt update\
sudo apt -y upgrade

Bước 2:\
Import Elasticsearch PGP Key
> sudo apt -y install gnupg\
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

Bước 3:\
Add APT repository

**For Elasticsearch 7.x (Latest)**
> sudo apt -y install apt-transport-https\
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

**For Elasticsearch 6.x**
> sudo apt -y install apt-transport-https\
echo "deb https://artifacts.elastic.co/packages/oss-6.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-6.x.list

Bước 4:
> sudo apt-get update && sudo apt-get install elasticsearch

Bước 5:

**Bật tắt Elasticsearch**

> sudo systemctl start elasticsearch.service\
sudo systemctl stop elasticsearch.service\
sudo systemctl enable elasticsearch.service\
sudo systemctl restart elasticsearch.service\
sudo systemctl status elasticsearch.service

```bash
$ systemctl status elasticsearch.service 
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/lib/systemd/system/elasticsearch.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-05-03 09:18:39 PDT; 18s ago
     Docs: http://www.elastic.co
 Main PID: 21459 (java)
    Tasks: 18 (limit: 1093)
   Memory: 429.0M
   CGroup: /system.slice/elasticsearch.service
           ├─21459 /usr/share/elasticsearch/jdk/bin/java -Xms512m -Xms512m -XX:+UseConcMarkSweepGC -XX:CMSIn
           └─21589 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

May 03 09:18:39 ubuntu systemd[1]: Started Elasticsearch.
```

`http://localhost:9200`

```json
{
  "name": "Cp8oag6",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "AT69_T_DTp-1qgIJlatQqA",
  "version": {
    "number": "7.10.2",
    "build_flavor": "default",
    "build_type": "tar",
    "build_hash": "f27399d",
    "build_date": "2016-03-30T09:51:41.449Z",
    "build_snapshot": false,
    "lucene_version": "8.7.0",
    "minimum_wire_compatibility_version": "1.2.3",
    "minimum_index_compatibility_version": "1.2.3"
  },
  "tagline": "You Know, for Search"
}
```

___

### Macos

> brew tap elastic/tap

> brew install elastic/tap/elasticsearch-full
___

### Window

***Đơn giản chỉ cần tải file .zip về hoặc file .msi giải nén ra và cài đặt, thao tác chỉ cần ấn next và ok là có thể cài
đặt được.***

[Link tải file .zip](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-windows-x86_64.zip)

[Link tải file .msi](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2.msi)
___

### Docker

***Khuyến nghị nên cài bằng docker vì 1 số lợi ích như sau:***

1. Dễ dàng cài đặt hoặc gỡ bỏ
2. Cài đặt được nhiều phiên bản elastic cùng 1 máy
3. Tự động hóa cài đặt kèm các ứng dụng đi kèm (thiết lập cấu hình, và các thao tác tự động, sao lưu dữ liệu)

Bước 1:\
Pull docker image từ docker hub về:
> docker pull docker.elastic.co/elasticsearch/elasticsearch:7.10.2

Bước 2:\
> docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.10.2

* Với 2 bước đơn giản như trên là đã có thể cài đặt elasticsearch thành công!
* Nhưng để thấy lợi ích to lớn hơn khi cài đặt bằng Docker ta có thể cài bằng docker-compose với nhiều tùy chọn và thiết
  lập hơn.

Hướng dẫn cài đặt:

1. Tạo 1 file **docker-compose.yml** sau với nội dung như bên dưới
2. Tại thư mục chứa file docker-compose bật terminal tại đó
3. Chạy lệnh `docker-compose up -d` để cài đặt
4. Đợi quá trình cài đặt hoàn tất

**docker-compose.yml** - es7

```yml
version: "3.3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    container_name: elasticsearch6
    restart: always
    volumes:
      #- ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./elasticsearch:/usr/share/elasticsearch/data
    environment:
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
      - http.port=9200
      - http.cors.enabled=true
      - http.cors.allow-origin=http://localhost:1358,http://127.0.0.1:1358,https://opensource.appbase.io
      - http.cors.allow-headers=X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
      - http.cors.allow-credentials=true
      - "xpack.security.enabled=false"
    ports:
      - 9300:9300
      - 9200:9200
    networks:
      - net_es
  kibana:
    image: docker.elastic.co/kibana/kibana:7.5.2
    container_name: kibana6
    restart: always #unless-stopped
    environment:
      - "pack.monitoring.collection.enabled=false"
      - "xpack.watcher.enabled=false"
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
    networks:
      - net_es
networks:
  net_es:
    driver: bridge
```
`docker images`

|REPOSITORY|TAG|IMAGE ID|CREATED|SIZE|
|:-------|:------|:-------|:-------|:-------|
|docker.elastic.co/elasticsearch/elasticsearch|7.10.2|0b58e1cea500|7 days ago|814MB|
|docker.elastic.co/kibana/kibana|7.5.2|a6e894c36481|12 months ago|950MB|

`docker ps -a`

|CONTAINER ID|IMAGE|COMMAND|CREATED|STATUS|PORTS|NAMES|
|:-------|:------|:-------|:-------|:------|:-------|:-------|
|708098b2693c|docker.elastic.co/elasticsearch/elasticsearch:7.10.2|"/tini -- /usr/local…"|5 days ago|Exited (255) 14 hours ago|0.0.0.0:9400->9200/tcp, 0.0.0.0:9500->9300/tcp|elasticsearch7|
|e05979cfa5cc|docker.elastic.co/kibana/kibana:6.0.1|"/bin/bash /usr/loca…"|7 days ago|Up 14 hours|0.0.0.0:5601->5601/tcp|kibana7|

### Cài đặt thêm

**Cài đặt kibana trên Docker**
Lợi ích:
  * Dùng để thực hiện query elastic search dễ dàng hơn

Pull docker image từ docker hub về:
>docker pull docker.elastic.co/kibana/kibana:7.10.2

>docker run --link YOUR_ELASTICSEARCH_CONTAINER_NAME_OR_ID:elasticsearch -p 5601:5601 docker.elastic.co/kibana/kibana:7.10.2

Cài đặt thành công kiểm tra tại đường dẫn: [http://localhost:5601/](http://localhost:5601/)

**Cài đặt Dejavu trên Docker**

Lợi ích:
* Hỗ trợ query dữ liệu trên elasticsearch
  * Hiển thị dữ liệu được đánh index
* Có giao diện mẫu để thực hiện search

GIT: https://github.com/appbaseio/dejavu
>docker run -p 1358:1358 -d appbaseio/dejavu

Cài đặt thành công kiểm tra tại đường dẫn: [http://localhost:1358/](http://localhost:1358/)

**Các công cụ bổ trợ**

[Elasticsearch head](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm) dùng để query dữ liệu, hiển thị dữ liệu index, ...

Postman dùng để test dữ liệu
