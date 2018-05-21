---
title: "Kubernetes Helm"
date: 2018-05-18T12:26:43+08:00
draft: true
---

## 安装 Helm
kubectl create -f tiller-rbac.yml
helm init -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.0 --service-account tiller
helm fetch --untar stable/prometheus
cd prometheus

## 安装 traefik，配置 letsencrypt 泛域名 HTTPS 证书

### 创建一个 kube secret
里面存放 cloudflare 的 CLOUDFLARE_EMAIL 和 CLOUDFLARE_API_KEY (Global API Key)
然后用类似下面的语法引用

env:
    - name: CLOUDFLARE_EMAIL
      valueFrom:
        secretKeyRef:
            name: cloudflare
            key: CLOUDFLARE_EMAIL
    - name: CLOUDFLARE_API_KEY
      valueFrom:
        secretKeyRef:
            name: cloudflare
            key: CLOUDFLARE_API_KEY
### 创建ConfigMap
kubectl create configmap traefik-conf --from-file=traefik.toml -n kube-system