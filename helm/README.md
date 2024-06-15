# 使用自定义指标



### Prometheus

添加仓库

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
```

部署Prometheus生态组件

```bash
helm install prometheus prometheus-community/prometheus --namespace monitoring --values prom-values.yaml --create-namespace
```



### Prometheus Adapter

部署prometheus-adapter

```bash
helm install prometheus-adapter prometheus-community/prometheus-adapter --values prom-adapter-values.yaml --namespace monitoring
```



Prometheus Adapter通过一组“发现（discovery）”规则（rules）来确定要公开哪些指标，以及如何公开这些指标。每条规则都是独立执行的（因此要确保规则彼此互斥），它通常包含需要由Adpater在API 公开指标时所需采取的多个个步骤。

每条规则大致可以分为四个部分：

> ***发现\***：指定Adpater如何找到此规则的所有 Prometheus 指标；
>
> ***关联\***：指定适Adpater应如何确定特定指标与哪些 Kubernetes 资源相关联；
>
> ***命名\***：指定Adpater应如何在自定义指标 API 中公开指标；
>
> ***查询\***：指定如何将对一个或多个 Kubernetes 对象上的特定指标的请求转换为对 Prometheus 的查询；

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





### 测试

待Prometheus和Prometheus Adapter的相关Pod均就绪后，获取针对现存系统环境由规则生成的自定义指标信息。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

部署示例应用metrcis app，它附带有“http_requests_total”指标。

```bash
```

待Metrics App的Pod就绪后，等待Prometheus Server的几个指标抓取周期，即可尝试获取由规则生成自定义指标http_requests_per_second。

```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second | jq .
```




