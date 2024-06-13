# Deploy Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
helm install prometheus prometheus-community/prometheus --namespace monitoring --values prom-values.yaml --create-namespace

helm install prometheus-adapter prometheus-community/prometheus-adapter --values prom-adapter-values.yaml --namespace monitoring --create-namespace

kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second | jq .
