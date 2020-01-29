# Prometheus

## 部署架构：

- Kubernetes version: v1.16.3
- 部署架构：







## 第一步：创建NameSpace："monitoring"

`$ kubectl apply -f prometheus-ns.yml`

所有Prometheus相关的对象都部署在NameSpace“monitoring”之下。



## 第二步：部署node-exporter

node-exporter用于提供*NIX内核的硬件以及系统指标，采集服务器层面的运行指标，包括机器的loadavg、filesystem、meminfo等。

### 1. 部署node-exporter daemonset

`$ kubectl apply -f node-exporter-daemonset.yml`

- 原始docker image是prom/node-exporter:v0.14.0
- 类型是daemonset，每个node上都会部署一份
- 使用宿主机网络，开放宿主机端口9100，可以直接通过<node_ip>:9100访问，如下：

![image-20200129144239797](/var/folders/tm/tv6_sl5s3r57lgzq3_rtn5mh0000gn/T/abnerworks.Typora/image-20200129144239797.png)

![image-20200129144332417](/var/folders/tm/tv6_sl5s3r57lgzq3_rtn5mh0000gn/T/abnerworks.Typora/image-20200129144332417.png)

### 2. 部署node-exporter service

`$ kubectl apply -f node-exporter-service.yml`

- 使用Cluster_IP，不能直接访问



## 第三步：部署kube-state-metrics

kube-state-metrics关注于获取k8s各种资源的最新状态，如deployment或者daemonset，将k8s的运行状况在内存中做个快照，并且获取新的指标。

### 1. 创建对应的ServiceAccount

`$ kubectl apply -f kube-state-metrics-ServiceAccount.yml`

### 2. 部署kube-state-metrics deployment

`$ kubectl apply -f kube-state-metrics-deploy.yml`

- 原始docker image是gcr.io/google_containers/kube-state-metrics:v0.5.0

### 3. 部署kube-state-metrics service

`$ kubectl apply -f kube-state-metrics-service.yml`

- 使用Cluster_IP，不能直接访问


## 第四步：部署node disk monitor

`$ kubectl apply -f monitor-node-disk-daemonset.yml`

- 监视Node的磁盘占用情况
- 原始docker image是giantswarm/tiny-tools:latest和dockermuenster/caddy:0.9.3
- 类型是daemonset，每个node上都会部署一份



## 第五步：部署Prometheus

### 1. 创建对应的ServiceAccount

`$ kubectl apply -f prometheus-k8s-ServiceAccount.yml`

### 2. 创建配置相关的configmap

`$ kubectl apply -f prometheus-config-configmap.yml`

可以到https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config查看相关设置的定义

### 3. 创建警告规则相关的configmap

`$ kubectl apply -f prometheus-rules-configmap.yml`

### 4. 创建Prometheus的缺省用户及密码

`$ kubectl apply -f prometheus-secret.yml`

- 缺省用户/密码为admin/admin： `$ echo "YWRtaW4=" | base64 -D`

### 5. 部署Prometheus Server的Deployment

`$ kubectl apply -f prometheus-deploy.yml`

- 原始docker image是prom/prometheus:v1.7.0

- 指向了alertmanager的service：`-alertmanager.url=http://alertmanager:9093/`

### 6. 部署Prometheus Server的Service

`$ kubectl apply -f prometheus-service.yml`

- 类型是NodePort，由K8s自由分配，通过 `$ kubectl get service -n monitoring` 可以查到分配的端口

- 可以通过<Node_IP>:<Node_Port>访问Prometheus的界面：

![image-20200129202707383](/var/folders/tm/tv6_sl5s3r57lgzq3_rtn5mh0000gn/T/abnerworks.Typora/image-20200129202707383.png)

- 在Alerts，可以看到根据rule定义的警报规则报出的告警信息：

![image-20200129202940375](/var/folders/tm/tv6_sl5s3r57lgzq3_rtn5mh0000gn/T/abnerworks.Typora/image-20200129202940375.png)

- 在Status->Targets，可以看到根据配置设置的监视目标：

![image-20200129203213531](/var/folders/tm/tv6_sl5s3r57lgzq3_rtn5mh0000gn/T/abnerworks.Typora/image-20200129203213531.png)



## 第六步：部署Graphana



