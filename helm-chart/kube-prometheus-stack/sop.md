# SOP

## 前置作業
1. 安裝Helm
2. 新增prometheus-commnunity/kube-prometheus-stack helm repo
3. kubectl label nodes duncank8s-worker3 appkind=gitlab

## 設定步驟
1. 產生預設設定檔
    ```bash
    helm show values prometheus-community/kube-prometheus-stack > prometheus-values.yaml
    ```

2. **Alertmanager部分**
- Replica
    ```yaml
    replicas: 2
    ```
- Storage
    ```yaml
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: standard
          accessModes: ["ReadWriteOnce"]
          resources:
          requests:
            storage: 50Gi
    ```
- nodeSelector
    ```yaml
    nodeSelector:
      appkind: gitlab
    ```

3. **Grafana部分**
- 在namespaceOverride下方新增nodeSelector
  ```yaml
  nodeSelector:
    appkind: gitlab
  ```
- TimeZone
  ```yaml
  defaultDashboardsTimezone: Asia/Taipei
  ```
- Default Password
  ```yaml
  adminPassword: admin
  ```
- Storage
  ```yaml
  persistence:
    type: pvc
    enabled: true
    storageClassName: standard
    accessModes:
      - ReadWriteOnce
    size: 10Gi
  ```

4. **KubeletService部分**
- Namespace
  ```yaml
  namespace: monitoring
  ```

5. **PrometheusSpec部分**
- NodeSelector
  ```yaml
  nodeSelector:
    appkind: gitlab
  ```
- Storage
  ```yaml
  storageSpec:
    ## Using PersistentVolumeClaim
    volumeClaimTemplate:
      spec:
        storageClassName: standard
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 50Gi
  ```

## 注意地方
- 下述component透過helm cahrt部署時預設都會在kube-system namespace底下建立service，service名稱為{helm-chart-deploy-name}-componentname
    * kube-prometheus-coredns
    * kube-prometheus-kube-controller-manager
    * kube-prometheus-kube-etcd
    * kube-prometheus-kube-proxy
    * kube-prometheus-kube-scheduler

- 如果不想在Namespace: kube-system底下建立上述Service可以設定取消部署

  1. **Kubelet Component部分**
  - Namespace
    ```yaml
    namespace: monitoring
    ```

  2. **kubeControllerManager Component部分**
  - Service
    ```yaml
    service:
      enabled: false
    ```

  3. **CoreDNS Component部分**
  - Enable
    ```yaml
    enabled: false
    ```

  4. **KubeEtcd Component部分**
  - Service
    ```yaml
    service:
      enabled: false
    ```

  5. **KubeScheduler Component部分**
  - Service
    ```yaml
    service:
      enabled: false
    ```

  6. **KubeProxy Component部分**
  - Service
    ```yaml
    service:
      enabled: false
    ```
- 必要Image
  1. alertmanagerSpec
      ```yaml
      image:
          repository: quay.io/prometheus/alertmanager
          tag: v0.24.0
          sha: ""
      ```
  2. grafana
    - 在persistence之後新增
      ```yaml
      image:
        repository: grafana/grafana
        tag: 8.4.4-ubuntu
        sha: ""
        pullPolicy: IfNotPresent
      testFramework:
        image: "bats/bats"
        tag: "v1.6.0"
        imagePullPolicy: IfNotPresent
      downloadDashboardsImage:
        repository: curlimages/curl
        tag: 7.83.1
        sha: ""
      initChownData:
        image:
          repository: busybox
          tag: "stable"
          sha: ""
          pullPolicy: IfNotPresent
      ```
    - 修改sidecar
      ```yaml
      sidecar:
        image:
          repository: quay.io/kiwigrid/k8s-sidecar
          tag: 1.17.0
          sha: ""
          imagePullPolicy: IfNotPresent
      ```
  3. kubeStateMetrics
    - 在kube-state-metrics下方新增
      ```yaml
      kube-state-metrics:
        image:
          repository: bitnami/kube-state-metrics
          tag: 2.4.2
          pullPolicy: IfNotPresent
      ```
  4. nodeExporter
    - 在prometheus-node-exporter下方新增
      ```yaml
      prometheus-node-exporter:
        image:
          repository: quay.io/prometheus/node-exporter
          tag: "v1.3.1"
          pullPolicy: IfNotPresent
      ```
  5. prometheusOperator
    - 修改admissionWebhooks
      ```yaml
        image:
          repository: jettech/kube-webhook-certgen
          tag: v1.5.2
          sha: ""
          pullPolicy: IfNotPresent
      ```
    - 修改image
      ```yaml
        image:
          repository: quay.io/prometheus-operator/prometheus-operator
          tag: v0.55.0
          sha: ""
          pullPolicy: IfNotPresent
      ```
    - 修改prometheusConfigReloader
      ```yaml
        image:
          repository: quay.io/prometheus-operator/prometheus-config-reloader
          tag: v0.55.0
          sha: ""
      ```
    - 修改thanosImage
      ```yaml
        repository: quay.io/thanos/thanos
          tag: v0.25.2
          sha: ""
      ```
    - 修改prometheusSpec
      ```yaml
        image:
          repository: quay.io/prometheus/prometheus
          tag: v2.34.0
          sha: ""
      ```