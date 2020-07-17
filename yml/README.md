> Kubernetes（以下简称k8s)是自动化容器操作的开源平台，这些操作包括部署，调度和节点集群间扩展。如果你曾经用过Docker容器技术部署容器，那么可以将Docker看成Kubernetes内部使用的低级别组件。Kubernetes不仅仅支持Docker，还支持Rocket，这是另一种容器技术。


#### 为什么使用k8s
- 自动化容器的部署和复制
- 随时扩展或收缩容器规模
- 将容器组织成组，并且提供容器间的负载均衡
- 很容易地升级应用程序容器的新版本
- 提供容器弹性，如果容器失效就替换它，等等...

![Kubernetes架构图](http://www.intecsec.com/wp-content/uploads/2020/06/k8s.png)

实际上，使用Kubernetes只需一个部署文件，使用一条命令就可以部署多层容器（前端，后台等）的完整集群：
```
$ kubectl create -f single-config-file.yaml
```

### 重要概念
- Cluster: Cluster是计算、存储和网络资源的集合，Kubernernets利用这些资源运行各种基于容器的应用。
- Master：Master是Cluster的大脑，它的主要职责是调度，即决定将应用放在哪里运行。
- Node：Node的职责是运行容器应用。Node由Master管理，Node负责监控并汇报容器的状态，同时根据Master的要求管理容器的生命周期。
- Pod：Pod是Kubernetes的最小工作单元。每个Pod包含一个多多个容器。

Kubernetes集群里有三种IP地址，分别如下：

- Node IP：Node节点的IP地址，即物理网卡的IP地址。
- Pod IP：Pod的IP地址，即docker容器的IP地址，此为虚拟IP地址。
- Cluster IP：Service的IP地址，此为虚拟IP地址。

[hello-minikube](https://kubernetes.io/docs/tutorials/hello-minikube/)

### kubectl 常用命令

```
运行应用
$ sudo kubectl apply -f *.yml

删除应用
$ sudo kubectl delete -f *.yml

进入k8s管理的一个容器
$ sudo kubectl get pods 
$ sudo kubectl exec -it peter-openjdk-5f6d8f8b5d-28rw4 /bin/bash
如一个pod里有多个容器,用--container or -c 参数。
$ sudo kubectl exec -it peter-producer-consumer -c peter-consumer sh

打标签：
[admin@k8s-master ~]$ sudo kubectl label node k8s-node2 disktype=kd2

指定部署到某节点
spec:
  nodeSelector:
    disktype: kd2
    
删除标签
[admin@k8s-master ~]$ sudo kubectl label node  k8s-node2  disktype-     

配置dns
[admin@k8s-master ~]$ sudo kubectl edit cm -n kube-system coredns

创建命名空间
kubectl get ns 
kubectl create ns trade-apps

[admin@k8s-master ~]$ sudo kubectl get nodes
[admin@k8s-master ~]$ sudo kubectl get deployments
[admin@k8s-master ~]$ sudo kubectl get pod --all-namespaces -o wide
[admin@k8s-master ~]$ sudo kubectl get pods -o wide
[admin@k8s-master ~]$ sudo kubectl get services
[admin@k8s-master ~]$ sudo kubectl get events
[admin@k8s-master ~]$ sudo kubectl describe node/deployment/pod/service/event

```

### 不同部署类型yml配置
kind的类型有：
- Deployment
- DaemonSet
- Job
- CronJob
- Pod
- Service

##### Service 
type有：
- ClusterIP  只能ClusterIP可以访问
- NodePort  可以提供外网访问Service
- LoadBalancer 均衡负载
```
apiVersion: v1
kind: Service
metadata:
  name: peter-nginx-svc
  namespace: default
spec:
  type: NodePort
  ports:
  - port: 8080 
    protocol: TCP
    nodePort: 3000 
    targetPort: 80  
  selector:
    run: peter-nginx
```
- port ClusterIP监听的端口
- nodePort NodePort监听的端口
- targetPort Pod监听的端口
- 选择labels为：peter-nginx的pod
- 将service的8080映射到pod的80端口，使用TCP协议
- pod < service < clusterIp（node机器ip) < 外网访问

##### Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: peter-openjdk
  namespace: default
spec:
  replicas: 2
  template:
    metadata:
      name: peter-java-base
      labels:
        app: peter-java-base
    spec:
      containers:
        - name: peter-java-base
          image: reg.was.ink/mcom/javabase-centos-openjdk:1.0
          imagePullPolicy: IfNotPresent
          command: [ "/bin/bash", "-ce", "tail -f /dev/null" ]
  selector:
    matchLabels:
      app: peter-java-base
```

##### Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: peter-producer-consumer
spec:
  containers:
  - name: peter-producer
    image: busybox
    volumeMounts:
      - mountPath: /producer_dir
        name: shared-volume
    args:
      - /bin/sh
      - -c
      - echo "hello world" > /producer_dir/hello ; sleep 30000

  - name: peter-consumer
    image: busybox
    volumeMounts:
      - mountPath: /consumer_dir
        name: shared-volume
    args:
      - /bin/sh
      - -c
      - cat /consumer_dir/hello ; sleep 30000

  volumes:
  - name: shared-volume
    emptyDir: {}
```
模拟生产者消费者，共享存储

##### 健康检查
```
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 35%
      maxUnavailable: 35%
          restartPolicy: Always
          readinessProbe:
            initialDelaySeconds: 120
            periodSeconds: 5
            httpGet:
              path: /actuator/health
              port: 10600
          livenessProbe:
            initialDelaySeconds: 120
            periodSeconds: 5
            httpGet:
              path: /actuator/health
              schema: HTTP
              port: 10600
```
和image参数同级别
schema: HTTP(默认) HTTPS
liveness 失败，容器重启
readiness失败，容器不可用，连续3次探测失败，会从负载均衡中移除
maxSurge: 35% 超过desired的上线 (1+35%) * replicas
maxUnavailable: 35%, 不可用占desired的最大比例  (1+35%) * replicas

##### k8s 配置文件 详解
```
apiVersion: v1 # 【必须】版本号
kind: Pod # 【必选】Pod
metadata: # 【必选-Object】元数据
  name: String # 【必选】 Pod的名称
  namespace: String # 【必选】 Pod所属的命名空间
  labels: # 【List】 自定义标签列表
    - name: String
  annotations: # 【List】 自定义注解列表
    - name: String
spec: # 【必选-Object】 Pod中容器的详细定义
  containers: # 【必选-List】 Pod中容器的详细定义
    - name: String # 【必选】 容器的名称
      image: String # 【必选】 容器的镜像名称
      imagePullPolicy: [Always | Never | IfNotPresent] # 【String】 每次都尝试重新拉取镜像 | 仅使用本地镜像 | 如果本地有镜像则使用，没有则拉取
      command: [String] # 【List】 容器的启动命令列表，如果不指定，则使用镜像打包时使用的启动命令
      args: [String] # 【List】 容器的启动命令参数列表
      workingDir: String # 容器的工作目录
      volumeMounts: # 【List】 挂载到容器内部的存储卷配置
        - name: String # 引用Pod定义的共享存储卷的名称，需使用volumes[]部分定义的共享存储卷名称
          mountPath: Sting # 存储卷在容器内mount的绝对路径，应少于512个字符
          readOnly: Boolean # 是否为只读模式，默认为读写模式
      ports: # 【List】 容器需要暴露的端口号列表
        - name: String  # 端口的名称
          containerPort: Int # 容器需要监听的端口号
          hostPort: Int # 容器所在主机需要监听的端口号，默认与containerPort相同。设置hostPort时，同一台宿主机将无法启动该容器的第二份副本
          protocol: String # 端口协议，支持TCP和UDP，默认值为TCP
      env: # 【List】 容器运行前需设置的环境变量列表
        - name: String # 环境变量的名称
          value: String # 环境变量的值
      resources: # 【Object】 资源限制和资源请求的设置
        limits: # 【Object】 资源限制的设置
          cpu: String # CPU限制，单位为core数，将用于docker run --cpu-shares参数
          memory: String # 内存限制，单位可以为MB，GB等，将用于docker run --memory参数
        requests: # 【Object】 资源限制的设置
          cpu: String # cpu请求，单位为core数，容器启动的初始可用数量
          memory: String # 内存请求，单位可以为MB，GB等，容器启动的初始可用数量
      livenessProbe: # 【Object】 对Pod内各容器健康检查的设置，当探测无响应几次之后，系统将自动重启该容器。可以设置的方法包括：exec、httpGet和tcpSocket。对一个容器只需要设置一种健康检查的方法
        exec: # 【Object】 对Pod内各容器健康检查的设置，exec方式
          command: [String] # exec方式需要指定的命令或者脚本
        httpGet: # 【Object】 对Pod内各容器健康检查的设置，HTTGet方式。需要指定path、port
          path: String
          port: Number
          host: String
          scheme: String
          httpHeaders:
            - name: String
              value: String
        tcpSocket: # 【Object】 对Pod内各容器健康检查的设置，tcpSocket方式
          port: Number
        initialDelaySeconds: Number # 容器启动完成后首次探测的时间，单位为s
        timeoutSeconds: Number  # 对容器健康检查的探测等待响应的超时时间设置，单位为s，默认值为1s。若超过该超时时间设置，则将认为该容器不健康，会重启该容器。
        periodSeconds: Number # 对容器健康检查的定期探测时间设置，单位为s，默认10s探测一次
        successThreshold: 0
        failureThreshold: 0
      securityContext:
        privileged: Boolean
  restartPolicy: [Always | Never | OnFailure] # Pod的重启策略 一旦终止运行，都将重启 | 终止后kubelet将报告给master，不会重启 | 只有Pod以非零退出码终止时，kubelet才会重启该容器。如果容器正常终止（退出码为0），则不会重启。
  nodeSelector: object # 设置Node的Label，以key:value格式指定，Pod将被调度到具有这些Label的Node上
  imagePullSecrets: # 【Object】 pull镜像时使用的Secret名称，以name:secretkey格式指定
    - name: String
  hostNetwork: Boolean # 是否使用主机网络模式，默认值为false。设置为true表示容器使用宿主机网络，不再使用docker网桥，该Pod将无法在同一台宿主机上启动第二个副本
  volumes: # 【List】 在该Pod上定义的共享存储卷列表
    - name: String # 共享存储卷的名称，volume的类型有很多emptyDir，hostPath，secret，nfs，glusterfs，cephfs，configMap
      emptyDir: {} # 【Object】 类型为emptyDir的存储卷，表示与Pod同生命周期的一个临时目录，其值为一个空对象：emptyDir: {}
      hostPath: # 【Object】 类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: String # Pod所在主机的目录，将被用于容器中mount的目录
      secret: # 【Object】类型为secret的存储卷，表示挂载集群预定义的secret对象到容器内部
        secretName: String
        items:
          - key: String
            path: String
      configMap: # 【Object】 类型为configMap的存储卷，表示挂载集群预定义的configMap对象到容器内部
        name: String
        items:
          - key: String
            path: String
```

imagePullPolicy属性
- Always 总是拉取镜像
- IfNotPresent 本地有则使用本地镜像,不拉取
- Never 只使用本地镜像，从不拉取，即使本地没有
- 如果省略imagePullPolicy 镜像tag为 :latest 策略为always ，否则 策略为 IfNotPresent


### 常见问题
#### 问题1：macbook安装kubernetes 报：Kubernenes is starting...
    1. 先拉去这个
    git clone https://github.com/maguowei/k8s-docker-for-mac
    2. 执行 
    ./load_images.sh
    3. 查看自己电脑装的Kubernetes的版本号
    4. 把2中拉下来的重新打标签
    k8s.gcr.io/kube-proxy                v1.15.5             cbd7f21fec99        4 months ago        82.4MB
    k8s.gcr.io/kube-apiserver            v1.15.5             e534b1952a0d        4 months ago        207MB
    k8s.gcr.io/kube-controller-manager   v1.15.5             1399a72fa1a9        4 months ago        159MB
    k8s.gcr.io/kube-scheduler            v1.15.5             fab2dded59dd        4 months ago        81.1MB
    
    $docker tag cbd7f21fec99 k8s.gcr.io/kube-proxy:自己电脑Kubernetes版本号
    
    5. reset > Reset Kubernetes cluster

#### 问题2：pod 报：Back-off restarting failed container
    在yml文件中，增加：
    command: [ "/bin/bash", "-ce", "tail -f /dev/null" ]

#### 问题3：mount.nfs: Stale file handle
服务端：
```
vim /etc/exports
systemctl restart nfs
```
客户端: 
```
umount /mnt/public
umount /mnt/public
vim /etc/fstab
mount -a
```

#### 问题4：dubbo rpc调用不通
需要在客户端上面启动openvpn
```
/etc/init.d/openvpn start
```


### 开启NFS服务器
yum -y install nfs-utils （只安装nfs-unitls）
yum install nfs-utils rpcbind -y （安装nfs-unitls和rpcbind服务，与上面二选一）
```
服务端：
$ vi /etc/exports
/public 192.168.245.1/24(ro)
/protected 192.168.245.1/24（rw）
systemctl reload nfs

客服端：
yum install -y nfs-utils
systemctl enable rpcbind.service
systemctl start rpcbind.service
[root@localhost ~]# vim /etc/fstab 
192.168.245.128:/public  /mnt/public     nfs    defaults  0 0
192.168.245.128:/protected /mnt/data     nfs    defaults  0 1
```

### 参考网站
- [Kubernetes 中文手册](https://www.kubernetes.org.cn/docs)
- [kubernetes 中文文档](http://docs.kubernetes.org.cn/)
- [kubernetes 官方中文文档](https://kubernetes.io/zh/docs/concepts/services-networking/service/)


### 微信公众号: 互联网技术的秘密 （intecsec）
![公众号](http://www.intecsec.com/wp-content/uploads/2020/06/intecsec_wechat.jpg)
### [官方网站: www.intecsec.com](http://www.intecsec.com)
### 个人微信号：sindoc
欢迎交流~