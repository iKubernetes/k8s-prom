部署Prometheus Adapater
==================

1. Prometheus Adapater的v0.8.4及之后版本的Image仅能通过 “registry.k8s.io/prometheus-adapter/prometheus-adapter:$VERSION”获取，之前版本的镜像，可到“quay.io/repository/coreos/k8s-prometheus-adapter-amd64” 仓库上查找。本示例中部署的是v0.10.0的版本。

2. 创建名为custom-metrics的名称空间。

   ```bash
   kubectl create namespace custom-metrics
   ```

3. 创建一个名为cm-adapter-serving-certs的secret对象，它需要具备serving.crt和serving.key两个键。These are the serving certificates used
   by the adapter for serving HTTPS traffic，更多的信息，请参考[auth concepts documentation](https://github.com/kubernetes-incubator/apiserver-builder/blob/master/docs/concepts/auth.md)。我们也可通过如下命令，运行目录中预置的gencerts.sh脚本进行创建。

   ```bash
   bash gencerts.sh
   ```

   该脚本会在manifests目录下创建一个cm-adapter-serving-certs.yaml的文件，它提供了相关secret对象的配置。

4. 运行如下命令，在custom-metrics名称空间中部署prometheus-adapter。

   ```bash
   kubectl apply -f manifests/
   ```

   > 提示：在部署prometheus-adpater之前，你可能需要事先修改ConfigMap资源，以添加需要暴露的自定义指标。
