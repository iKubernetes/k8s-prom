# 部署Prometheus支持自定义指标



### Prometheus

首先，添加Prometheus Community的Chart仓库。

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
```

运行如下命令，即可加载本地的values文件，部署Prometheus生态组件。本示例中，Prometheus的版本被限定在“v2”的版本上，使用的是“v2.55.1”的版本。若要移除该限制，注释掉prom-values.yaml文件中的“server.image.tag”即可。

```bash
helm install prometheus prometheus-community/prometheus --namespace monitoring --values prom-values.yaml --create-namespace
```

或者，也可以运行如下命令，直接加载在线的values文件，部署Prometheus生态组件。

```bash
helm install prometheus prometheus-community/prometheus --namespace monitoring \
          --values https://raw.githubusercontent.com/iKubernetes/k8s-prom/master/helm/prom-values.yaml --create-namespace
```

### Prometheus Adapter

#### 部署

运行如下命令，即可基于本地的values文件部署prometheus-adapter组件。

```bash
helm install prometheus-adapter prometheus-community/prometheus-adapter --values prom-adapter-values.yaml --namespace monitoring
```

或者，也可以运行如下命令，直接加载在线的values文件，部署Prometheus Adapter组件。

```bash
helm install prometheus-adapter prometheus-community/prometheus-adapter --namespace monitoring \
          --values https://raw.githubusercontent.com/iKubernetes/k8s-prom/master/helm/prom-adapter-values.yaml 
```

#### 规则配置说明

Prometheus Adapter通过一组“发现（discovery）”规则（rules）来确定要公开哪些指标，以及如何公开这些指标。每条规则都是独立执行的（因此要确保规则彼此互斥），它通常包含需要由Adpater在API 公开指标时所需采取的多个个步骤。

每条规则大致可以分为四个部分：

> **发现**：指定Adpater如何找到此规则的所有 Prometheus 指标；
>
> **关联**：指定适Adpater应如何确定特定指标与哪些 Kubernetes 资源相关联；
>
> **命名**：指定Adpater应如何在自定义指标 API 中公开指标；
>
> **查询**：指定如何将对一个或多个 Kubernetes 对象上的特定指标的请求转换为对 Prometheus 的查询；

下面是一个示例：

```yaml
    - seriesQuery: 'http_requests_total{kubernetes_namespace!="",kubernetes_pod_name!=""}'
      resources:
        overrides:
          kubernetes_namespace: {resource: "namespace"}
          kubernetes_pod_name: {resource: "pod"}
      name:
        matches: "^(.*)_total"
        as: "${1}_per_second"
      metricsQuery: rate(<<.Series>>{<<.LabelMatchers>>}[1m])
```



更详细的信息，请参考[官方文档](https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/config.md)。



### 测试

待Prometheus和Prometheus Adapter的相关Pod均就绪后，获取针对现存系统环境由规则生成的自定义指标信息。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

部署示例应用metrcis app，它附带有“http_requests_total”指标。

```bash
kubectl apply -f https://raw.githubusercontent.com/iKubernetes/k8s-prom/master/prometheus-adpater/example-metrics/metrics-example-app.yaml
```

待Metrics App的Pod就绪后，等待Prometheus Server的几个指标抓取周期，即可尝试获取由规则生成自定义指标http_requests_per_second。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second | jq .
```



### Blackbox Exporter

部署Blackbox Exporter，启用黑盒监控；如下命令指定了Release名称为“prometheus-blackbox-exporter”，该名称作为Service名称，将被下面示例中的配置引用。

```bash
helm install --name-template prometheus-blackbox-exporter  prometheus-community/prometheus-blackbox-exporter \
          -f blackbox-exporter-values.yaml -n monitoring
```

随后，需要在Prometheus的values文件中，启用额外的Scrape Job，以关联Blackbox Exporter。一个示例配置如下，注意该字段的值必须为字符型数据，因此需要在字段后添加“|”。

```yaml
extraScrapeConfigs: |
  - job_name: 'prometheus-blackbox-exporter'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://www.magedu.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: prometheus-blackbox-exporter:9115
```

最后，更新Prometheus的Release，确保配置生效。
