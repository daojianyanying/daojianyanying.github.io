---
title: k8s的service-ingress
date: 2025-08-11 15:42:58
update: 2025-08-12 15:42:58
tags: [k8s, nginx]
index_img: /img/bg/ingress.jpg
excerpt: k8s的ingress搭建
sticky: 100
category: k8s
---
一、Ingress出现的原因
1. Service只能提供4层负载均衡，无法提供7层负载均衡
2. Service的访问方式是固定的，需要通过ClusterIP进行访问，而Ingress可以自定义访问方式，例如通过域名进行访问
二、Ingress的原理
1. Ingress是Kubernetes集群中的一个API对象，用于管理集群中的HTTP和HTTPS路由规则
2. Ingress Controller是一个独立的组件，用于实现Ingress的规则，例如Nginx Ingress Controller
3. Ingress Controller会监听Ingress对象的变化，并根据Ingress规则生成对应的Nginx配置文件
4. 当客户端请求到达Ingress Controller时，Ingress Controller会根据Nginx配置文件将请求转发到对应的Service
三、Ingress的安装
1. 安装Nginx Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/baremetal/deploy.yaml
```
2. 验证安装
```bash
kubectl get pods -n ingress-nginx
```
四、Ingress的使用
1. 创建一个Ingress对象
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations: