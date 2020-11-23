

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch-master
  namespace: elasticsearch-prod
spec:
  selector:
    matchLabels:
      component: elasticsearch-master
  replicas: 3
  serviceName: "elasticsearch-master"
  template:
    metadata:
      labels:
        component: elasticsearch-master
    spec:
      hostAliases:
      - ip: "10.0.0.80"
        hostnames:
        - "el-stats1.yad2.hfa"
      - ip: "10.0.0.81"
        hostnames:
        - "el-stats2.yad2.hfa"
      - ip: "10.0.0.82"
        hostnames:
        - "el-stats3.yad2.hfa"
      terminationGracePeriodSeconds: 10
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: component
                operator: In
                values:
                - elasticsearch
            topologyKey: kubernetes.io/hostname
      initContainers:
      - name: configure-sysctl
        securityContext:
          runAsUser: 0
          privileged: true
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
      containers:
      - name: elasticsearch-master
        securityContext:
          privileged: false
          capabilities:
            add:
              - IPC_LOCK
              - SYS_RESOURCE
        image: quay.io/pires/docker-elasticsearch-kubernetes:5.4.0
        imagePullPolicy: Always
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: "CLUSTER_NAME"
          value: "analytics"
        - name: "NUMBER_OF_MASTERS"
          value: "2"
        - name: "NODE_MASTER"
          value: "true"
        - name: "NODE_INGEST"
          value: "false"
        - name: NODE_DATA
          value: "false"
        - name: HTTP_ENABLE
          value: "true"
        - name: "NETWORK_HOST"
          value: "0.0.0.0"
        - name: "MAX_LOCAL_STORAGE_NODES"
          value: "1"
        - name: "MEMORY_LOCK"
          value: "false"
        - name: "HTTP_CORS_ENABLE"
          value: "false"
        - name: "ES_JAVA_OPTS"
          value: "-Xms4g -Xmx4g"
        - name: "HTTP_CORS_ALLOW_ORIGIN"
          value: "false"
        - name: "DISCOVERY_SERVICE"
          value: "el-stats1.yad2.hfa, el-stats2.yad2.hfa, el-stats3.yad2.hfa, el-stats4.yad2.hfa, elasticsearch-master-0-internal, elasticsearch-master-1-internal, elasticsearch-master-2-internal"
        ports:
        - containerPort: 9300
          name: transport
          protocol: TCP
        volumeMounts:
        - name: config-volume
          mountPath: /elasticsearch/config/elasticsearch.yml
          subPath: elasticsearch.yaml
        # - name: storage
        #   mountPath: /data
        - mountPath: "/data"
          name: elastic-pvc
        resources:
          requests:
            memory: "8000Mi"
            cpu: "2"
          limits:
            memory: "8000Mi"
            cpu: "2"
      volumes:
          - name: config-volume
            configMap:
              name: elasticsearch-config
          # - emptyDir:
          #     medium: ""
          #   name: "storage"
          - name: elastic-pvc
            persistentVolumeClaim:
              claimName: elastic-pvc
  volumeClaimTemplates:
  - metadata:
      name: elastic-pvc
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 20Gi
      selector:
        matchLabels:
          app: elastic
      storageClassName: slow
      volumeMode: Filesystem
```


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    app: elastic
  name: elastic-pv1
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 20Gi
  nfs:
    path: /volumes/disk2/k8s-pv/elastic1
    server: yad2-img1
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    app: elastic
  name: elastic-pv2
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 20Gi
  nfs:
    path: /volumes/disk2/k8s-pv/elastic2
    server: yad2-img1
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    app: elastic
  name: elastic-pv3
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 20Gi
  nfs:
    path: /volumes/disk2/k8s-pv/elastic3
    server: yad2-img1
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  volumeMode: Filesystem
```