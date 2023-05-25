# k8s-prom

- prometheus：部署Promethues Metrics API Server所需要的各资源配置清单。
- prometheus-adapter：部署基于prometheus的自定义指标API服务器所需要的各资源配置清单。
- podinfo：测试使用的podinfo相关的deployment和service对象的资源配置清单。
- node_exporter：于kubernetes集群各节点部署node_exporter。
- kube-state-metrics：聚合kubernetes资源对象，提供指标数据。
- alertmanager：部署AlertManager告警系统。

### 部署Prometheus

部署Prometheus监控系统

```bash
kubectl apply -f namespace.yaml
kubectl apply -f prometheus/ -n prom
```

部署node-exporter

```bash
kubectl apply -f node-exporter/
```

### 部署Kube-State-Metrics

部署kube-state-metrics，监控Kubernetes集群的服务指标。

```bash
kubectl apply -f kube-state-metrics/
```

### 部署AlertManager

部署AlertManager，为Prometheus-Server提供可用的告警发送服务。

```bash
kubectl apply -f alertmanager/
```

### 部署Prometheus Adpater

参考相关目录中的[README](prometheus-adpater/README.md)文件中的部署说明。

## iKubernetes公众号

![ikubernetes公众号二维码](https://github.com/iKubernetes/Kubernetes_Advanced_Practical_2rd/raw/main/imgs/iKubernetes%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

## 《Kubernetes进阶实战第2版》

- [淘宝直达](https://s.taobao.com/search?q=kubernetes%E8%BF%9B%E9%98%B6%E5%AE%9E%E6%88%98%E7%AC%AC2%E7%89%88&imgfile=&commend=all&ssid=s5-e&search_type=item&sourceId=tb.index&spm=a21bo.2017.201856-taobao-item.1&ie=utf8&initiative_id=tbindexz_20170306)
- [京东商城直达](https://search.jd.com/Search?keyword=kubernetes%E8%BF%9B%E9%98%B6%E5%AE%9E%E6%88%98%E7%AC%AC2%E7%89%88&enc=utf-8&suggest=2.def.0.base&wq=kubernetes%E8%BF%9B%E9%98%B6%E5%AE%9E%E6%88%98&pvid=286ff777931e4075a762f321a0fb1139)
- [当当直达](http://search.dangdang.com/?key=kubernetes%BD%F8%BD%D7%CA%B5%D5%BD%B5%DA%B6%FE%B0%E6&act=input)

![图书封面](https://github.com/iKubernetes/Kubernetes_Advanced_Practical_2rd/raw/main/imgs/book.jpg)


## 版权声明
本文档由[马哥教育](www.magedu.com)开发，允许自由转载，但必须保留马哥教育及相关的一切标识。另外，商用需要征得马哥教育的书面同意。


### 参考文献

- https://github.com/stefanprodan/k8s-prom-hpa

