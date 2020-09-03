原文链接：https://my.oschina.net/yimingkeji/blog/2978993

---

## 1. 安装ElasticSearch

拉取镜像，选择版本为6.5.0。

```bash
$ docker pull elasticsearch:6.5.0
```

查看镜像。

```bash
$ docker images
```

![2020-06-09-Ncj6HS](https://image.ldbmcs.com/2020-06-09-Ncj6HS.jpg)

启动一个容器。

```bash
$ docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -p 9200:9200 -p 9300:9300 elasticsearch:6.5.0
42d639a089348b393b0cc912141ef357b6565996c5c4863e363f8729da229d7d
```

然后访问 `GET localhost:9200` ，发现未启动成功，查看日志。

```bash
$ docker logs -f 42d6
[2018-12-05T06:07:06,546][INFO ][o.e.b.BootstrapChecks    ] [IHubvTB] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2018-12-05T06:07:06,630][INFO ][o.e.n.Node               ] [IHubvTB] stopping ...
[2018-12-05T06:07:06,939][INFO ][o.e.n.Node               ] [IHubvTB] stopped
[2018-12-05T06:07:06,939][INFO ][o.e.n.Node               ] [IHubvTB] closing ...
[2018-12-05T06:07:06,973][INFO ][o.e.n.Node               ] [IHubvTB] closed
```

这里提示：`vm.max_map_count [65530] is too low, increase to at least [262144]`，说**max_map_count**的值太小了，需要设大到262144。

查看max_map_count的值。

```bash
$ cat /proc/sys/vm/max_map_count
65530
```

重新设置max_map_count的值。

```bash
$ sysctl -w vm.max_map_count=262144
vm.max_map_count = 262144
```

再次启动容器。

```bash
$ docker start 42d6
```

再次访问 `GET localhost:9200`。

```json
{
    "name": "IHubvTB",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "edN3o1PkTAKF5C4N-Uw7tQ",
    "version": {
        "number": "6.5.0",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "816e6f6",
        "build_date": "2018-11-09T18:58:36.352602Z",
        "build_snapshot": false,
        "lucene_version": "7.5.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```

安装成功。

## 2. head插件安装

### 2.1 5.x版本head插件安装说明

1.x版本和2.x版本，可以直接使用plugin命令来安装head插件

5.x以上版本，无法通过plugin命令来安装。

https://github.com/mobz/elasticsearch-head 文档说明：

插件不支持，需要借助node来跑一个服务。

```bash
Running as a plugin of Elasticsearch (deprecated)
for Elasticsearch 5.x: site plugins are not supported. Run as a standalone server
for Elasticsearch 2.x: sudo elasticsearch/bin/plugin install mobz/elasticsearch-head
for Elasticsearch 1.x: sudo elasticsearch/bin/plugin -install mobz/elasticsearch-head/1.x
for Elasticsearch 0.x: sudo elasticsearch/bin/plugin -install mobz/elasticsearch-head/0.9
open http://localhost:9200/_plugin/head/
```

```bash
Running with built in server
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
open http://localhost:9100/
This will start a local webserver running on port 9100 serving elasticsearch-head
```

或者安装google插件。

```bash
Running as a Chrome extension
Install ElasticSearch Head from the Chrome Web Store.
Click the extension icon in the toolbar of your web browser.
```

![2020-06-09-R8DM1k](https://image.ldbmcs.com/2020-06-09-R8DM1k.jpg)

但是还有一种安装方式，就是通过docker安装，访问地址是` http://localhost:9100/`

```bash
Running with docker
for Elasticsearch 5.x: docker run -p 9100:9100 mobz/elasticsearch-head:5
for Elasticsearch 2.x: docker run -p 9100:9100 mobz/elasticsearch-head:2
for Elasticsearch 1.x: docker run -p 9100:9100 mobz/elasticsearch-head:1
for fans of alpine there is mobz/elasticsearch-head:5-alpine
open http://localhost:9100/
```

### 2.2 使用docker安装head插件

拉取镜像（这里选择国内镜像源）。

```bash
$ docker pull mobz/elasticsearch-head
```

启动容器。

```bash
$ docker run -d -p 9100:9100 --name elasticsearch-head elasticsearch-head
```

然后浏览器访问 `localhost:9200`。

![2020-06-09-9T4LFU](https://image.ldbmcs.com/2020-06-09-9T4LFU.jpg)

出现这个界面，说明head插件安装成功。

但是发现健康值为：未连接？

打开浏览器调试，发现报错信息：

```bash
Access to XMLHttpRequest at 'http://localhost:9200/' from origin 'http://localhost:9100' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

这里也就是**跨域错误**。

https://github.com/mobz/elasticsearch-head 文档中也有说明：

```bash
Enable CORS in elasticsearch
When not running as a plugin of elasticsearch (which is not even possible from version 5) you must enable CORS in elasticsearch otherwise your browser will rejects requests which appear insecure.

In elasticsearch configuration;

add http.cors.enabled: true
you must also set http.cors.allow-origin because no origin allowed by default. http.cors.allow-origin: "*" is valid value, however it’s considered as a security risk as your cluster is open to cross origin from anywhere.
```

## 3. ElasticSearch跨域设置

进入elasticsearch容器。

```bash
$ docker exec -it 42d639a08934 /bin/bash
```

找到配置文件的目录。

```bash
$ /usr/share/elasticsearch/config
$ ls
elasticsearch.keystore  ingest-geoip  log4j2.properties  roles.yml  users_roles
elasticsearch.yml       jvm.options   role_mapping.yml   users
```

修改`elasticsearch.yml`，需要安装vim（参考 https://my.oschina.net/yimingkeji/blog/2978974）

如果提示apt-get不存在，使用yum安装vim。

```bash
$ yum install -y vim
```

安装后编辑elasticsearch.yml。

```bash
$ vim elasticsearch.yml
# ----添加内容----
# head插件设置
http.cors.enabled: true
http.cors.allow-origin: "*"
```

重启容器。

```bash
$ docker restart 42d639a08934
```

再次访问插件。

![2020-06-09-d6YGI8](https://image.ldbmcs.com/2020-06-09-d6YGI8.jpg)

安装完成。