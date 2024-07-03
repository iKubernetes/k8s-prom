# 自定义指标流水线及HPAv2

### 添加自定义规则

Prometheus Adapter需要基于定义的规则来生成自定义指标。例如，要将示例应用Metrics APP上的指标http_requests_total转为Prometheus上可用的指标http_requests_per_second，需要类似如下规则的支撑。

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

本示例中的ConfigMap配置文件“custom-metrics-config-map.yaml”在默认配置的基础上添加了如上配置，若部署prometheus-adapter无其它配置变化，则直接运行如下命令即可完成添加自定义的规则。
```bash
kubectl apply -f custom-metrics-config-map.yaml
```

### 测试HPA

首先部署Metrics APP示例于指定的名称空间，例如default。
```bash
kubectl apply -f metrics-example-app.yaml -n default
```

等待几个指标抓取周期，即可从Metrics APP的Pod上查看到相关的指标信息。
```bash
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second | jq .
```

生成的结果类似如下所示，其中的“metricName”为指标名称，“value”为相关的值，“100m”代表平均每秒钟接收到的请求数为0.1个。
```json
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": [
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "metrics-app-56c77b4999-f4l5q",
        "apiVersion": "/v1"
      },
      "metricName": "http_requests_per_second",
      "timestamp": "2024-07-01T11:13:08Z",
      "value": "100m",
      "selector": null
    },
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "metrics-app-56c77b4999-tf8zc",
        "apiVersion": "/v1"
      },
      "metricName": "http_requests_per_second",
      "timestamp": "2024-07-01T11:13:08Z",
      "value": "100m",
      "selector": null
    }
  ]
}
```


然后部署HPA的定义于同一名称空间。
```bash
kubectl apply -f metrics-app-hpa.yaml
```

生成的结果类似如下所示，其中的“TARGETS”字段中，左侧为当前的指标值，右侧为执行扩容的标准值。
```
NAME              REFERENCE                TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
metrics-app-hpa   Deployment/metrics-app   100m/5    2         10        2          22s
```

