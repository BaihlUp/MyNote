---
title: 基于Istio ingress对外提供服务
date: 2024-03-10
categories:
  - 虚拟化&云计算
tags:
  - 云原生
  - Istio
published: true
---
## 1 简介
上一篇是在Service Mesh中两个应用之间互相访问，本片介绍下通过Istio Ingress把服务网络中的应用对外暴露，通过Ingress Gateway控制从外边访问服务网格中的应用。
在Service Mesh中部署两个服务，一个httpserver应用，一个Nginx，通过在Ingress Gateway中配置路由控制，使用同一个域名，不同的URI访问两个应用服务。如下：
```
curl -H "Host: simple.baihl.io" $INGRESS_IP/simple/httpserver
curl -H "Host: simple.baihl.io" $INGRESS_IP/simple/nginx
```
`$INGRESS_IP` 为 Istio中Ingress Gateway对外的IP地址。
## 2 创建应用服务
```sh
kubectl create ns simple
kubectl create -f simple.yaml -n simple
kubectl create -f nginx.yaml -n simple
kubectl apply -f istio-specs.yaml -n simple
```
- simple.yaml

simple.yaml就是运行httpserver，并且定义了Service对外提供服务。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple
spec:
  replicas: 1
  selector:
    matchLabels:
      app: simple
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "80"
      labels:
        app: simple
    spec:
      containers:
        - name: simple
          imagePullPolicy: Always
          image: cncamp/httpserver:v1.0-metrics
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: simple
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: simple
```
- nginx.yaml
一个基本的Nginx服务
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: nginx
```
## 3 创建VirtualService
- istio-specs.yaml

VirtualService是在Istio服务网格内对服务的请求进行路由控制。
如下，通过创建VirtualService与gateway关联。分配控制对httpserver和nginx的访问。
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: simple
spec:
  gateways:
    - simple
  hosts:
    - simple.baihl.io
  http:
  # 会在Envoy中生成对应的route配置
  - match:
    - uri:
        exact: "/simple/httpserver"
    rewrite:
      uri: "/hello"
    route:
      - destination:
          host: simple.simple.svc.cluster.local
          port:
            number: 80
  - match:
    - uri:
        prefix: "/simple/nginx"
    rewrite:
      uri: "/"
    route:
      - destination:
          host: nginx.simple.svc.cluster.local
          port:
            number: 80
---
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: simple
spec:
  selector:
    istio: ingressgateway
  servers:
  # 会在Envoy中生成对应的listener配置
    - hosts:
        - simple.baihl.io
      port:
        name: http-simple
        number: 80 #对外提供的服务端口为80
        protocol: HTTP
```
部署完以上的服务，整体看下所有创建出来的对象：
执行`kubectl get pod,svc,deployment,endpoints,VirtualService,Gateway  -n simple`
![](https://s2.loli.net/2022/09/25/gcsn8SVxtMembPy.png)
可以看到，service/nginx 和 service/simple 分别是我们创建的两个应用程序Pod对外提供服务的Service，还有一个virtualservice，匹配的Host为"simple.baihl.io"。

## 4 通过Ingress Gateway访问服务
查看Ingress Gateway对外提供暴露的IP地址：
![](https://s2.loli.net/2022/09/25/zCrtjFTPSBX3cRD.png)
Ingress Gateway对外的IP为：`10.96.190.18`
```
INGRESS_IP=10.96.190.18
curl -H "Host: simple.baihl.io" $INGRESS_IP/simple/httpserver
curl -H "Host: simple.baihl.io" $INGRESS_IP/simple/nginx
```
- 访问httpserver

![](https://s2.loli.net/2022/09/25/JjTr8tv2Cc9Lk6s.png)
- 访问Nginx

![](https://s2.loli.net/2022/09/25/OidG6THwlRUCvfN.png)
## 5 分析请求过程
### 5.1 Ingress Gateway Service
在访问IngressGateway时，首先经过的就是Ingress Gateway配置的Service，可以看下Service配置是什么。
执行`kubectl get svc istio-ingressgateway -n istio-system  -o json`
```json
"ports": [
              {
                "name": "http2",
                "nodePort": 31373,
                "port": 80,
                "protocol": "TCP",
                "targetPort": 8080
            },
            {
                "name": "https",
                "nodePort": 31860,
                "port": 443,
                "protocol": "TCP",
                "targetPort": 8443
            },
        ],
        "selector": {
            "app": "istio-ingressgateway",
            "istio": "ingressgateway"
        },
        "sessionAffinity": "None",
        "type": "LoadBalancer"
```
Service使用的type为LoadBalancer，配置了对外提供的端口80映射到宿主机的31373，在上边访问应用服务的时候因为是在Master节点，所以就直接访问了Service的IP地址10.96.190.18，但是在集群外部是无法访问10.96.190.18的，需要访问Master节点对外的IP地址，对于我本地的环境来说就是Master节点的IP地址192.168.170.137。所以在集群外部访问httpserver和nginx，需要访问 10.96.190.18:31373，操作如下：
```sh
$ curl -H "Host: simple.baihl.io" 192.168.170.137:31373/simple/nginx
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0  90667      0 --:--:-- --:--:-- --:--:--  150k<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
如上我在本地笔记本上访问Master节点的IP地址请求Isito中的Nginx服务。
### 5.2 iptables规则
为什么在外部访问192.168.170.137:31373和在Master节点上访问IngressGateway的IP都可以获得应用服务的响应，就是因为在Kubernetes的Master节点上的iptables规则。
比如，访问192.168.170.137:31373
```sh
#对于访问目标端口为31373的请求，转到KUBE-SVC-G6D3V5KS3PXPUEDS
-A KUBE-NODEPORTS -p tcp -m comment --comment "istio-system/istio-ingressgateway:http2" -m tcp --dport 31373 -j KUBE-SVC-G6D3V5KS3PXPUEDS
# 匹配KUBE-SVC-G6D3V5KS3PXPUEDS请求转到KUBE-SEP-LAV34IONK7IOT5SV
-A KUBE-SVC-G6D3V5KS3PXPUEDS -m comment --comment "istio-system/istio-ingressgateway:http2" -j KUBE-SEP-LAV34IONK7IOT5SV
# 匹配KUBE-SEP-LAV34IONK7IOT5SV，请求做DNAT，目标IP修改为10.10.1.4，目标端口为8080
-A KUBE-SEP-LAV34IONK7IOT5SV -p tcp -m comment --comment "istio-system/istio-ingressgateway:http2" -m tcp -j DNAT --to-destination 10.10.1.4:8080
```
比如，访问10.96.190.18:80，会匹配到如下的iptables规则：
```sh
#对于访问目标IP为10.96.190.18/32，目标端口为80的请求，转到KUBE-SVC-G6D3V5KS3PXPUEDS
-A KUBE-SERVICES -d 10.96.190.18/32 -p tcp -m comment --comment "istio-system/istio-ingressgateway:http2 cluster IP" -m tcp --dport 80 -j KUBE-SVC-G6D3V5KS3PXPUEDS
# 匹配KUBE-SVC-G6D3V5KS3PXPUEDS请求转到KUBE-SEP-LAV34IONK7IOT5SV
-A KUBE-SVC-G6D3V5KS3PXPUEDS -m comment --comment "istio-system/istio-ingressgateway:http2" -j KUBE-SEP-LAV34IONK7IOT5SV
# 匹配KUBE-SEP-LAV34IONK7IOT5SV，请求做DNAT，目标IP修改为10.10.1.4，目标端口为8080
-A KUBE-SEP-LAV34IONK7IOT5SV -p tcp -m comment --comment "istio-system/istio-ingressgateway:http2" -m tcp -j DNAT --to-destination 10.10.1.4:8080
```
对比上边两种访问情况的iptables规则匹配流程可以看到，最终都是对请求做了DNAT后，请求的目标IP地址变为10.10.1.4，目标端口变为8080。
![](https://s2.loli.net/2022/09/25/Tf8bV7RmwKqPper.png)
可以看到，10.10.1.4的IP就是istio-ingressgateway的IP地址。
### 5.3 Envoy配置
目前请求转到了10.10.1.4:8080，所以可以看下在istio-ingressgateway中监听8080端口的是谁？
![](https://s2.loli.net/2022/09/25/5HcoTFPB7zNbW9Z.png)
使用nsenter命令，进入istio-ingressgateway内部查看，可以发现监听8080端口的正式Envoy程序。
下边就可以通过看Envoy的配置，看看请求后边是怎么处理的。
1. 首先看下监听8080端口的配置信息
```
baihl@baihl-master:~$ istioctl pc listener -n istio-system istio-ingressgateway-576c469c96-spndl --port 8080
ADDRESS PORT MATCH DESTINATION
0.0.0.0 8080 ALL   Route: http.8080
```
监听8080的listener匹配的route为http.8080，继续查看route信息。
2. 查看route为http.8080
```yaml
baihl@baihl-master:~$ istioctl pc route  -n istio-system istio-ingressgateway-576c469c96-spndl --name=http.8080 -o yaml
- ignorePortInHostMatching: true
  name: http.8080
  validateClusters: false
  virtualHosts:
  - domains:
    - simple.baihl.io
    includeRequestAttemptCount: true
    name: simple.baihl.io:80
    routes:
      match:
        caseSensitive: true
        ## 匹配/simple/httpserver的请求
        path: /simple/httpserver
      route:
      # httpserver route对应的cluster
        cluster: outbound|80||simple.simple.svc.cluster.local
        maxStreamDuration:
          grpcTimeoutHeaderMax: 0s
          maxStreamDuration: 0s
        prefixRewrite: /hello
    - decorator:
        operation: nginx.simple.svc.cluster.local:80/simple/nginx*
      match:
        caseSensitive: true
        prefix: /simple/nginx
      metadata:
        filterMetadata:
          istio:
            config: /apis/networking.istio.io/v1alpha3/namespaces/simple/virtual-service/simple
      route:
      # nginx 服务对应的cluster
        cluster: outbound|80||nginx.simple.svc.cluster.local
        maxStreamDuration:
          grpcTimeoutHeaderMax: 0s
          maxStreamDuration: 0s
        prefixRewrite: /
```
根据route配置可以看到：
1. 请求uri为/simple/httpserver，被outbound|80||simple.simple.svc.cluster.local处理。
2. 请求uri为/simple/nginx，被outbound|80||nginx.simple.svc.cluster.local处理。
3. 查看cluster

下边查看httpserver对应的的cluter：outbound|80||simple.simple.svc.cluster.local：
```sh
baihl@baihl-master:~$ istioctl pc endpoints  -n istio-system istio-ingressgateway-576c469c96-spndl --cluster="outbound|80||simple.simple.svc.cluster.local"
ENDPOINT          STATUS      OUTLIER CHECK     CLUSTER
10.10.1.13:80     HEALTHY     OK                outbound|80||simple.simple.svc.cluster.local
```
可以看到httpserver最终选择的endpoint为10.10.1.13:80。
```sh
baihl@baihl-master:~$ kubectl get pod -n simple -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP           NODE         NOMINATED NODE   READINESS GATES
nginx-deployment-85b98978db-9rhvd   1/1     Running   0          110m   10.10.1.14   baihl-node   <none>           <none>
simple-fb6498fdb-rmdg2              1/1     Running   0          110m   10.10.1.13   baihl-node   <none>           <none>
```
可以看到运行httpserver的simple Pod，在集群内的IP地址就是10.10.1.13。
同样的方式，可以查看nginx服务对应的cluster：outbound|80||nginx.simple.svc.cluster.local：
```
baihl@baihl-master:~$ istioctl pc endpoints  -n istio-system istio-ingressgateway-576c469c96-spndl --cluster="outbound|80||nginx.simple.svc.cluster.local"
ENDPOINT          STATUS      OUTLIER CHECK     CLUSTER
10.10.1.14:80     HEALTHY     OK                outbound|80||nginx.simple.svc.cluster.local
```
10.10.1.14对应的就是nginx-deployment Pod在集群内的IP地址。
根据上边的分析，在集群外通过istio-ingressgateway访问集群内部的应用程序，经过了iptables规则和ingressgateway中的路由控制，最后把请求分发到各个应用程序，通过ingressgateway也可以实现更加复杂的流量管理需求。

