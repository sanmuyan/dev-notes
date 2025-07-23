# K8s SkyWalking 部署指南

## skywalking

注意此方式部署的 pg 没有持久存储

```shell
export SKYWALKING_RELEASE_VERSION=4.7.0
export SKYWALKING_RELEASE_NAME=skywalking
export SKYWALKING_RELEASE_NAMESPACE=skywalking-system

kubectl create namespace "${SKYWALKING_RELEASE_NAMESPACE}"

helm install "${SKYWALKING_RELEASE_NAME}" \
  oci://registry-1.docker.io/apache/skywalking-helm \
  --version "${SKYWALKING_RELEASE_VERSION}" \
  -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  --set oap.image.tag=10.1.0 \
  --set oap.storageType=postgresql \
  --set ui.image.tag=10.1.0 \
  --set elasticsearch.enabled=false \
  --set postgresql.enabled=true \
  --set oap.dynamicConfig.enabled=false \
  --set oap.replicas=1
```

## skywalking-rover

监控 K8s Service 网络，部署完后可以在 skywalking-ui 中查看 kubernetes Sercvice 流量和调用关系

```shell
kubectl apply -f skywalking-rover.yaml -n skywalking-system
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: skywalking-rover
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: skywalking-rover
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: skywalking-rover
subjects:
  - kind: ServiceAccount
    name: skywalking-rover
    namespace: skywalking-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: skywalking-rover
rules:
  - apiGroups: [""]
    resources: ["pods", "nodes", "services"]
    verbs: ["get", "watch", "list"]
---

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: skywalking-rover
spec:
  selector:
    matchLabels:
      name: skywalking-rover
  template:
    metadata:
      labels:
        name: skywalking-rover
    spec:
      serviceAccountName: skywalking-rover
      serviceAccount: skywalking-rover
      containers:
        - name: skywalking-rover
          image: apache/skywalking-rover:0.7.0
          securityContext:
            capabilities:
              add:
                - SYS_PTRACE
                - SYS_ADMIN
            privileged: true
          volumeMounts:
            - name: host
              mountPath: /host
              readOnly: true
            - name: sys
              mountPath: /sys
              readOnly: true
          env:
            - name: ROVER_ACCESS_LOG_ACTIVE
              value: "true"
            - name: ROVER_PROCESS_DISCOVERY_KUBERNETES_ACTIVE
              value: "true"
            - name: ROVER_PROCESS_DISCOVERY_KUBERNETES_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: ROVER_BACKEND_ADDR
              value: skywalking-skywalking-helm-oap:11800
            - name: ROVER_HOST_MAPPING
              value: /host
      hostPID: true
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      volumes:
        - name: host
          hostPath:
            path: /
            type: Directory
        - name: sys
          hostPath:
            path: /sys
            type: Directory
```

## otel-collector

部署 opentelemetry-collector 采集 K8s 集群数据，部署完后可以在 skywalking-ui 中查 K8s 集群状态

```shell
kubectl apply -f otel-collector.yaml -n skywalking-system
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: otel-collector
  namespace: skywalking-system
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: otel-collector
rules:
  - apiGroups: [""]
    resources: ["pods", "endpoints", "services", "nodes", "nodes/metrics"]
    verbs: ["get", "watch", "list"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: otel-collector
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: otel-collector
subjects:
  - kind: ServiceAccount
    name: otel-collector
    namespace: skywalking-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-conf
  labels:
    app: opentelemetry
    component: otel-collector-conf
data:
  otel-collector-config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: ${env:MY_POD_IP}:4317
          http:
            endpoint: ${env:MY_POD_IP}:4318
    processors:
      batch:
      memory_limiter:
        # 80% of maximum memory up to 2G
        limit_mib: 1500
        # 25% of limit up to 2G
        spike_limit_mib: 512
        check_interval: 5s
    extensions:
      zpages: {}
    exporters:
      otlp:
        endpoint: "http://someotlp.target.com:4317" # Replace with a real endpoint.
        tls:
          insecure: true
    service:
      extensions: [zpages]
      pipelines:
        traces/1:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [otlp]
---          
apiVersion: v1
kind: Service
metadata:
  name: otel-collector
  labels:
    app: opentelemetry
    component: otel-collector
spec:
  ports:
  - name: otlp-grpc # Default endpoint for OpenTelemetry gRPC receiver.
    port: 4317
    protocol: TCP
    targetPort: 4317
  - name: otlp-http # Default endpoint for OpenTelemetry HTTP receiver.
    port: 4318
    protocol: TCP
    targetPort: 4318
  - name: metrics # Default endpoint for querying metrics.
    port: 8888
  selector:
    component: otel-collector
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  labels:
    app: opentelemetry
    component: otel-collector
spec:
  selector:
    matchLabels:
      app: opentelemetry
      component: otel-collector
  minReadySeconds: 5
  progressDeadlineSeconds: 120
  replicas: 1 #TODO - adjust this to your own requirements
  template:
    metadata:
      labels:
        app: opentelemetry
        component: otel-collector
    spec:
      serviceAccountName: skywalking-rover
      serviceAccount: otel-collector
      containers:
      - command:
          - "/otelcol"
          - "--config=/conf/otel-collector-config.yaml"
        image: otel/opentelemetry-collector:latest
        name: otel-collector
        resources:
          limits:
            cpu: 1
            memory: 2Gi
          requests:
            cpu: 200m
            memory: 400Mi
        ports:
        - containerPort: 55679 # Default endpoint for ZPages.
        - containerPort: 4317 # Default endpoint for OpenTelemetry receiver.
        - containerPort: 14250 # Default endpoint for Jaeger gRPC receiver.
        - containerPort: 14268 # Default endpoint for Jaeger HTTP receiver.
        - containerPort: 9411 # Default endpoint for Zipkin receiver.
        - containerPort: 8888  # Default endpoint for querying metrics.
        env:
          - name: MY_POD_IP
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: status.podIP
          - name: GOMEMLIMIT
            value: 1600MiB
        volumeMounts:
        - name: otel-collector-config-vol
          mountPath: /conf
#        - name: otel-collector-secrets
#          mountPath: /secrets
      volumes:
        - configMap:
            name: otel-collector-conf
            items:
              - key: otel-collector-config
                path: otel-collector-config.yaml
          name: otel-collector-config-vol
#        - secret:
#            name: otel-collector-secrets
#            items:
#              - key: cert.pem
#                path: cert.pem
#              - key: key.pem
#                path: key.pem
```

修改 `configmap/otel-collector-conf`，然后重启 deployment

```yaml
receivers:
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-cadvisor'
          scheme: https
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
            - role: node
          relabel_configs:
            - action: labelmap
              regex: __meta_kubernetes_node_label_(.+)
            - source_labels: [ ]
              target_label: cluster
              replacement: skywalking-showcase
            - target_label: __address__
              replacement: kubernetes.default.svc:443
            - source_labels: [ __meta_kubernetes_node_name ]
              regex: (.+)
              target_label: __metrics_path__
              replacement: /api/v1/nodes/$${1}/proxy/metrics/cadvisor
            - source_labels: [ instance ]
              separator: ;
              regex: (.+)
              target_label: node
              replacement: $$1
              action: replace
        # @feature: kubernetes-monitor; configuration to scrape Kubernetes Endpoints metrics
        - job_name: kube-state-metrics
          metrics_path: /metrics
          kubernetes_sd_configs:
            - role: endpoints
          relabel_configs:
            - source_labels: [ __meta_kubernetes_service_label_app_kubernetes_io_name ]
              regex: kube-state-metrics
              replacement: $$1
              action: keep
            - action: labelmap
              regex: __meta_kubernetes_service_label_(.+)
            - source_labels: [ ]
              target_label: cluster
              replacement: skywalking-showcase

  otlp:
    protocols:
      grpc:
        endpoint: ${env:MY_POD_IP}:4317
      http:
        endpoint: ${env:MY_POD_IP}:4318
processors:
  batch:
  memory_limiter:
    # 80% of maximum memory up to 2G
    limit_mib: 1500
    # 25% of limit up to 2G
    spike_limit_mib: 512
    check_interval: 5s
extensions:
  zpages: {}
exporters:
  otlp:
    endpoint: "http://skywalking-skywalking-helm-oap:11800" # Replace with a real endpoint.
    tls:
      insecure: true
service:
  extensions: [zpages]
  pipelines:
    traces/1:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
    metrics:
      receivers: [prometheus]
      exporters: [otlp]    
```