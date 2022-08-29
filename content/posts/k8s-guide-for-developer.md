---
title: "Kubernetes Guide for Developer"
date: 2022-03-18T14:32:45+08:00
draft: true
tags: ["Kubernetes", "Ingress"]
---
## Environment
Before you try to use any client tool connect to Kubernetes, please check the endpoint field 
"server" of "cluster" credential config file.

Please make sure you're able connect with the kube-apiserver.

For instance, example below shown here is "k8s-cluster"
```yaml
apiVersion: v1
kind: Config
clusters:
  - name: team-a-admin@kubernetes
    cluster:
      server: 'https://k8s-cluster:8443'
      certificate-authority-data: >-
        xxxxx
users:
  - name: team-a-admin
    user:
      token: >-
        xxxxx
contexts:
  - name: team-a-admin@kubernetes
    context:
      user: team-a-admin
      cluster: team-a-admin@kubernetes
      namespace: team-a
current-context: team-a-admin@kubernetes
```

This example is based on self-host Kubernetes v1.18 environment, you machine might not be able to recognize the 
target,

it lack of DNS and k8s apiserver certificate support currently.

If you have trouble with this kind of problem, please add a record in my <b>hosts</b> file, 
that makes my machine's dns resolvable.

```txt
# File path
# Windows: `C:\Windows\System32\drivers\etc\hosts`
# Linux: `/etc/hosts`
192.168.24.1 k8s-cluster`
```

#### K8S client
* https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
* https://k8slens.dev/
* https://k9scli.io
* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Docker Registry
```bash
$ docker login my-registry.com
Username: 
Password:
$ docker push my-registry.com/my-app:0.0.1
```
## Deployment example in our environment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: team-a
spec:
  selector:
    matchLabels:
      app: my-app
  replicas: 1 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-registry/my-app:0.0.1
        ports:
        - containerPort: 80
      nodeSelector:
        node-role.kubernetes.io/worker: 'true' # We have many VMs in the Kubernetes cluster. 
We prefer all the pod running on certain VM, please add description below in my deployment. 
That will make pods running on our pre labeled K8S worker node.
      imagePullSecrets:
        - name: my-registry-auth
```
* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

## Service, Ingress
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  namespace: team-a
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: my-app
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: team-a
  labels:
    app: my-app
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
  rules:
    - host: my-app.deltaww.com
      http:
        paths:
          - path: /prefix(/|$)(.*)
            backend:
              serviceName: my-app-svc
              servicePort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-domain-ingress
  namespace: team-a
  labels:
    app: my-domain
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
  tls:
    - hosts:
        - my-domain.com
      secretName: my-tls # please make sure there's an existed K8S secret in 
the same namespace
  rules:
    - host: my-domain.com
      http:
        paths:
          - path: /prefix(/|$)(.*)
            backend:
              serviceName: my-app-svc
              servicePort: 80
```

* https://kubernetes.io/docs/concepts/services-networking/service/
* https://kubernetes.io/docs/concepts/services-networking/ingress/

## PVC
Here's a PVC resource example.

The storageClassName describe below, rook-ceph is installed in our self-host environment. If you application is running on other Kubernetes cluster, please contact administrator.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-logs-pvc
  namespace: team-a
  labels:
    app: my-app-logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: rook-ceph-block
  volumeMode: Filesystem
```
Here's an example for logging file path mounting PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app2
  namespace: team-a
spec:
  volumes:
    - name: my-app-logs-storage
      persistentVolumeClaim:
        claimName: my-app-logs-pvc
  containers:
    - name: my-app2
      image: my-registry.com/my-app:0.0.1
      volumeMounts:
        - mountPath: "/app/logs"
          name: my-app-logs-storage
  nodeSelector:
    node-role.kubernetes.io/worker: 'true' # We have many VMs in the Kubernetes cluster. We 
prefer all the pod running on certain VM, please add description below in my deployment. 
That will make pods running on our pre labeled K8S worker node.
  imagePullSecrets:
    - name: my-registry-auth
```

* https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
