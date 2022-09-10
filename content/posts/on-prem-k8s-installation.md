---
title: "On-Premises K8s Installation"
date: 2020-09-18T10:00:04+08:00
draft: true
tags: ["Kubernetes", "CSI", "CNI"]
---
![kubeadm-ha-topology](https://d33wubrfki0l68.cloudfront.net/d1411cded83856552f37911eb4522d9887ca4e83/b94b2/images/kubeadm/kubeadm-ha-topology-stacked-etcd.svg)
#### Load balancer & VIP
[HAProxy & Keepalived](https://www.digitalocean.com/community/tutorials/how-to-set-up-highly-available-haproxy-servers-with-keepalived-and-reserved-ips-on-ubuntu-14-04)
#### Init HA control plane & worker node
```bash
sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
...
...
You can now join any number of control-plane node by running the following command on each as a root:
    kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use kubeadm init phase upload-certs to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:
    kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```
#### Networking with Weave Net
```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
* https://www.weave.works/docs/net/latest/overview/
* https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
#### Ingress controller with Nginx
```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/cloud/deploy.yaml
```
* https://kubernetes.github.io/ingress-nginx/
#### Storage
* Ceph
![Rook Ceph](https://rook.io/docs/rook/v1.10/Getting-Started/ceph-storage/kubernetes.png)
```bash
$ git clone --single-branch --branch v1.10.1 https://github.com/rook/rook.git
$ cd rook/deploy/examples
$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml
$ kubectl create -f cluster.yaml
```
* https://rook.io/docs/rook/v1.10/Getting-Started/intro/
#### Monitoring
* Prometheus, Grafana
![Prometheus](https://prometheus.io/assets/architecture.png)
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
$ helm install -n [NAMESPACE] [RELEASE_NAME] prometheus-community/kube-prometheus-stack
```
* https://prometheus.io
* https://grafana.com/oss/grafana/
#### Log
* Loki
![Loki](https://grafana.com/static/img/logs/logs-loki-diagram.svg)
```bash
$ helm repo add grafana https://grafana.github.io/helm-charts
$ helm repo update
$ helm upgrade --install --namespace=[NAMESPACE] [RELEASE_NAME] grafana/loki-stack
```
* https://grafana.com/oss/loki/
