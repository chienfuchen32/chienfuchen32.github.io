---
title: "Nginx Ingress Controller K8S Guide"
date: 2022-04-15T15:45:07+08:00
draft: true
tags: ["Kubernetes", "Ingress", "reverse proxy"]
---
## Nginx 介紹
Nginx是非同步框架的網頁伺服器，也可以用作反向代理、負載平衡器和HTTP快取。
![nginx](https://miro.medium.com/max/1024/1*TrNJZqECEj0eVuJDeNKtNQ.png)
* 設置範例
```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    listen 443 ssl default_server;
    ssl_certificate /tls/server.crt;
    ssl_certificate_key /tls/server.key;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
		proxy_pass http://192.168.24.100:9090/;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "Upgrade";
		proxy_set_header Host $host;
		proxy_connect_timeout 60s;
		proxy_read_timeout 300s;
		proxy_send_timeout 300s;
    }


    location /socket.io {
        proxy_pass http://192.168.24.100:9091/;
        #Version 1.1 is recommended for use with keepalive connections
        proxy_http_version 1.1; #WebSocket
        proxy_set_header Upgrade $http_upgrade; #WebSocket
        proxy_set_header Connection $connection_upgrade; #WebSocket       
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Cookie $http_cookie;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```
可以透過`nginx -s reload`讓nginx對新的配置生效
* https://nginx.org/en/docs/beginners_guide.html
* https://nginx.org/en/docs/

## K8S service
![service](https://miro.medium.com/max/1094/1*YeBt0Z9BEA7T_tUO4iYQyQ.png)
K8S 定義 Service 這個 resource object，在 pod 的前方提供了一個抽象層，讓外部的服務可以用 
domain name 的方式存取 pod，而 domain name <–> Pod 這一段問題，就由 Service 來處理。 但利用 
domain name 來存取終究還是需要一個 IP address，而每個 Service 都會自帶 VIP，讓 network traffic 
有辦法正常送過來，並導到後方真正提供服務的 pod。
* https://kubernetes.io/docs/concepts/services-networking/service/
* https://godleon.github.io/blog/Kubernetes/k8s-Service-Overview/

## K8S Ingress
![ingress](https://miro.medium.com/max/1024/1*BYGsxY_GR-eZgNJfbbikzA.png)
Ingress是從Kubernetes叢集外部訪問到群集內部服務的方法之一
* https://kubernetes.io/docs/concepts/services-networking/ingress/

## Nginx Ingress controller
為了讓Ingress資源工作，叢集必須有一個正在運行的Ingress controller，其功能實作如:
* SSL/TLS offloading
* 反向代理，域名解析
* 四層和七層區別網路封包轉導

```
Kubernetes                                                  Workstation
+---------------------------------------------------+     +------------------+
|                                                   |     |                  |
|  +-----------+   apiserver        +------------+  |     |  +------------+  |
|  |           |   proxy            |            |  |     |  |            |  |
|  | apiserver |                    |  ingress   |  |     |  |  ingress   |  |
|  |           |                    | controller |  |     |  | controller |  |
|  |           |                    |            |  |     |  |            |  |
|  |           |                    |            |  |     |  |            |  |
|  |           |  service account/  |            |  |     |  |            |  |
|  |           |  kubeconfig        |            |  |     |  |            |  |
|  |           +<-------------------+            |  |     |  |            |  |
|  |           |                    |            |  |     |  |            |  |
|  +------+----+      kubeconfig    +------+-----+  |     |  +------+-----+  |
|         |<--------------------------------------------------------|        |
|                                                   |     |                  |
+---------------------------------------------------+     +------------------+
```
* https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
* 
https://www.f5points.com.tw/2021/10/27/ingress-controller-%E9%81%B8%E5%9E%8B%E6%8C%87%E5%8D%97%EF%BC%8C%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%EF%BC%9A%E7%A2%BA%E5%AE%9A%E9%9C%80%E6%B1%82/#:~:text=Ingress%20Controller%20%E6%98%AF%E4%B8%80%E7%A8%AE%E5%B0%88%E7%94%A8,%E9%80%B2%E5%85%A5%20Kubernetes%20%E9%9B%86%E7%BE%A4%E7%9A%84%E6%B5%81%E9%87%8F
* https://kubernetes.github.io/ingress-nginx/how-it-works/

## 常見使用情形範例與除錯方式
### DNS,IP,避免影響正式環境
```/etc/hosts
# 20220415 lab
192.168.24.1 lab-20220415.foo.com
192.168.24.1 lab-20220415.bar.foo.com
```
### Backend service (K8S pod or external service) 
* 於pod/container中使用`curl` / `wget` 工具:
```bash
$ curl localhost:5000/healthz
```
* 透過本機連線來除錯 service port forward (cli or lens)
```bash
$ kubectl port-forward -n 20220415-nginx-lab service/nginx-lab 25000/5000
```
* 
https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/
### rewrite
```bash
echo '
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - path: /something(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: http-svc
            port: 
              number: 80
' | kubectl create -f -
```
In this ingress definition, any characters captured by (.*) will be assigned to the 
placeholder $2, which is then used as a parameter in the rewrite-target annotation.
For example, the ingress definition above will result in the following rewrites:
1. `rewrite.bar.com/something` rewrites to `rewrite.bar.com`
2. `rewrite.bar.com/something/` rewrites to `rewrite.bar.com/`
3. `rewrite.bar.com/something/new` rewrites to `rewrite.bar.com/new`
```
example

rewrite.bar.com/something/new    |------------|      IP:PORT/new          |-----------------|
-------------------------------->|   Nginx    |-------------------------->| backend service |
                                 |------------|                           |-----------------|

```
https://kubernetes.github.io/ingress-nginx/examples/rewrite/
### SSL/TLS Offloading
* https://www.nginx.com/blog/nginx-ssl/

### Websocket
* https://nginx.org/en/docs/http/websocket.html
* 
https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#custom-timeouts
### TCP/UDP service
* https://nginx.org/en/docs/stream/stream_processing.html
* https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/

# lab and demo
* https://github.com/chienfuchen32/k8s-nginx-ingress-hands-on-lab
1. develope app
2. deploy
* examples
https://lab-20220415.foo.com/
https://lab-20220415.bar.foo.com
https://lab-20220415.foo.com/nginx-lab/
https://lab-20220415.foo.com/nginx-lab-rewrite/

# 特殊情境
## 若服務不在K8S中，在同一個LAN的VM，可以從K8S叢集連線的到
* 假設服務跑在http://192.168.24.101:5000
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-svc
  namespace: app
spec:
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: ClusterIP
  sessionAffinity: None
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-svc
  namespace: app
subsets:
  - addresses:
      - ip: 192.168.24.101
    ports:
      - port: 5000
        protocol: TCP
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: external-ingress
  namespace: app
  labels:
    app: external-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
  rules:
    - host: mydomain.com
      http:
        paths:
          - path: /path(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              serviceName: external-svc
              servicePort: 5000
```
