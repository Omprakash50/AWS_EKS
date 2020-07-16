# AWS_EKS


## Elastic Kubernetes Service
EKS stands for Elastic Kubernetes Service, which is an Amazon offering that helps in running the Kubernetes on AWS without requiring the user to maintain their own Kubernetes control plane. It is a fully managed service by Amazon.

## eksctl
eksctl is a simple CLI tool for creating clusters on EKS - Amazon's new managed Kubernetes service for EC2. It is written in Go, and uses CloudFormation. You can create a cluster in minutes with just one command – eksctl create cluster !

we can connect to AWS via 3 ways
- webui
- CLI
- API (Terraform)

### CLI

For this first we require aws configure to login to the aws cloud and then we need eksctl command which is like a client command only build for EKS Service.
This will login to the aws in mumbai data center

![m](cli_config.png)

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: sddcluster
  region: ap-south-1

nodeGroups:
  - name: ng1
    instanceType: t2.micro
    desiredCapacity: 2
    ssh:
      publicKeyName: Omos
  - name: ng2
    instanceType: t3.small
    desiredCapacity: 1
    ssh:
      publicKeyName: Omos 
```

```eksctl create cluster -f <clusterfile_name>```

```aws eks update-kubeconfig --name <cluster_name>```

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: efs-provisioner
spec:
  selector:
    matchLabels:
      app: efs-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: efs-provisioner
    spec:
      containers:
        - name: efs-provisioner
          image: quay.io/external_storage/efs-provisioner:v0.1.0
          env:
            - name: FILE_SYSTEM_ID
              value: xxxxxxxx
            - name: AWS_REGION
              value: ap-south-1
            - name: PROVISIONER_NAME
              value: efs-storage
          volumeMounts:
            - name: pv-volume
              mountPath: /persistentvolumes
      volumes:
        - name: pv-volume
          nfs:
            server: xxxxxxxxyyyyyyy.amazonaws.com
            path: /
```
```kubectl create -f file.yml```

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nfs-provisioner-role-binding
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
```kubectl create -f file.yml```

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: aws-efs
provisioner: efs-storage
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: efs-prometheus
  annotations:
    volume.beta.kubernetes.io/storage-class: "aws-efs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: efs-grafana
  annotations:
    volume.beta.kubernetes.io/storage-class: "aws-efs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
```kubectl create -f file.yml```

```
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  ports:
    - port: 80
  selector:
    app: prometheus
    tier: backend
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
      tier: backend
  strategy:
    type: Recreate
  template:
    metadata:
      labels: 
        app: prometheus
        tier: backend
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        volumeMounts:
        - name: pro-volume
          mountPath: /etc/prometheus
        ports:
        - containerPort: 80
          name: prometheus
      volumes:
        - name: all-info
          persistentVolumeClaim:
            claimName: efs-prometheus
```
```kubectl create -f file.yml```

```
apiVersion: v1
kind: Service
metadata:
  name: grafana
  labels:
    app: grafana
spec:
  ports:
    - port: 80
  selector:
    app: grafana
    tier: frontend
  type: LoadBalancer
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: grafana
  labels:
    app: grafana
    tier: frontend
spec:
  selector:
    matchLabels:
      app: grafana
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: grafana
        tier: frontend
    spec:
      containers:
      - image: grafana/grafana:latest
        name: grafana
        ports:
        - containerPort: 80
          name: grafana
        volumeMounts:
        - name: all-info
          mountPath: /var/lib/grafana
      volumes:
      - name: all-info
        persistentVolumeClaim:
          claimName: efs-grafana
```
```kubectl create -f file.yml```

```eksctl delete cluster sddcluster```

![m](prometheus_start.png)

___
![m](pro_start.png)

___
![m](prometheus.png)

___
![m](grafana_start.png)

___
![m](grafana.png)
