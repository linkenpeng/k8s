
### centos7-openjdk8-jre

构建一个基于CentOs的Java8运行时环境
JRE采用OpenJDK8U-jre_x64_linux_hotspot_8u242b08
[下载地址https://adoptopenjdk.net/releases.html](https://adoptopenjdk.net/releases.html)
##### 构建
```
docker build -t centos7.7-openjdk8-jre:1.1 .

```

##### 上传
```
docker push linkenpeng/centos7.7-openjdk8-jre:1.2
```

#### 使用
```
docker pull linkenpeng/centos7.7-openjdk8-jre:1.2
docker run -it linkenpeng/centos7.7-openjdk8-jre:1.2 /bin/bash
```

### centos7-openjdk8-aliyun
和centos7-openjdk8-jre一样的配置，区别是centos的yum源替换成了阿里云的源