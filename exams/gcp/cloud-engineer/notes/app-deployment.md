# Application Deployment - Cloud Engineer

## Compute Engine Deployment

### Instance Creation

**Creating VM Instances**:
```bash
# Create basic instance
gcloud compute instances create my-instance \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud

# Create instance with startup script
gcloud compute instances create web-server \
  --zone=us-central1-a \
  --machine-type=n1-standard-1 \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx'

# Create instance with custom metadata and scopes
gcloud compute instances create app-server \
  --zone=us-central1-a \
  --machine-type=n1-standard-2 \
  --scopes=storage-rw,logging-write,monitoring-write \
  --metadata=app-version=v1.2.0,environment=production \
  --tags=app-server,backend

# Create instance from custom image
gcloud compute instances create custom-instance \
  --zone=us-central1-a \
  --machine-type=n1-standard-1 \
  --image=my-custom-image \
  --image-project=my-project-id

# Create preemptible instance (cost savings)
gcloud compute instances create preemptible-worker \
  --zone=us-central1-a \
  --machine-type=n1-standard-2 \
  --preemptible
```

### Instance Templates

**Creating and Managing Templates**:
```bash
# Create instance template
gcloud compute instance-templates create web-template \
  --machine-type=n1-standard-1 \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --boot-disk-size=10GB \
  --tags=http-server,https-server \
  --metadata=startup-script-url=gs://my-bucket/startup.sh

# Create template with container
gcloud compute instance-templates create-with-container nginx-template \
  --container-image=gcr.io/my-project/nginx:latest \
  --container-restart-policy=always \
  --machine-type=n1-standard-1

# Create template with multiple network interfaces
gcloud compute instance-templates create multi-nic-template \
  --machine-type=n1-standard-2 \
  --network-interface=network=default,subnet=default \
  --network-interface=network=private-network,subnet=private-subnet

# List instance templates
gcloud compute instance-templates list

# Describe instance template
gcloud compute instance-templates describe web-template

# Delete instance template
gcloud compute instance-templates delete web-template
```

### Managed Instance Groups (MIGs)

**Creating MIGs**:
```bash
# Create managed instance group
gcloud compute instance-groups managed create web-mig \
  --base-instance-name=web \
  --template=web-template \
  --size=3 \
  --zone=us-central1-a

# Create regional MIG (higher availability)
gcloud compute instance-groups managed create web-mig-regional \
  --base-instance-name=web \
  --template=web-template \
  --size=6 \
  --region=us-central1

# Set autoscaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --zone=us-central1-a \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=90

# Set autoscaling based on load balancer utilization
gcloud compute instance-groups managed set-autoscaling web-mig \
  --zone=us-central1-a \
  --max-num-replicas=10 \
  --target-load-balancing-utilization=0.8

# Set named ports (for load balancing)
gcloud compute instance-groups managed set-named-ports web-mig \
  --named-ports=http:80,https:443 \
  --zone=us-central1-a
```

**Managing MIG Size**:
```bash
# Resize instance group
gcloud compute instance-groups managed resize web-mig \
  --size=5 \
  --zone=us-central1-a

# List instances in MIG
gcloud compute instance-groups managed list-instances web-mig \
  --zone=us-central1-a

# Delete specific instances (recreated automatically)
gcloud compute instance-groups managed delete-instances web-mig \
  --instances=web-abc1,web-abc2 \
  --zone=us-central1-a

# Abandon instances (remove from MIG without deleting)
gcloud compute instance-groups managed abandon-instances web-mig \
  --instances=web-abc3 \
  --zone=us-central1-a
```

### Rolling Updates and Canary Deployments

**Rolling Updates**:
```bash
# Perform rolling update with new template
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v2 \
  --zone=us-central1-a

# Rolling update with specific parameters
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v2 \
  --max-surge=3 \
  --max-unavailable=1 \
  --min-ready-sec=60 \
  --zone=us-central1-a

# Update with replacement method (recreate vs substitute)
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v2 \
  --replacement-method=RECREATE \
  --zone=us-central1-a
```

**Canary Deployments**:
```bash
# Canary deployment - 10% new version, 90% old version
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v1,target-size=90% \
  --version=template=web-template-v2,target-size=10% \
  --zone=us-central1-a

# Canary with absolute numbers
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v1 \
  --version=template=web-template-v2,target-size=2 \
  --zone=us-central1-a

# Check update status
gcloud compute instance-groups managed describe web-mig \
  --zone=us-central1-a

# Stop ongoing update
gcloud compute instance-groups managed rolling-action stop-proactive-update web-mig \
  --zone=us-central1-a
```

**Health Checks for MIGs**:
```bash
# Create health check
gcloud compute health-checks create http web-health-check \
  --port=80 \
  --request-path=/health \
  --check-interval=10s \
  --timeout=5s \
  --unhealthy-threshold=3 \
  --healthy-threshold=2

# Set health check for MIG
gcloud compute instance-groups managed set-autohealing web-mig \
  --health-check=web-health-check \
  --initial-delay=300 \
  --zone=us-central1-a

# Create HTTPS health check
gcloud compute health-checks create https secure-health-check \
  --port=443 \
  --request-path=/api/health

# Create TCP health check
gcloud compute health-checks create tcp tcp-health-check \
  --port=3306
```

## Google Kubernetes Engine (GKE) Deployment

### Cluster Creation

**Creating GKE Clusters**:
```bash
# Create basic cluster
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3

# Create cluster with specific machine type
gcloud container clusters create production-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=n1-standard-4 \
  --disk-size=50

# Create regional cluster (high availability)
gcloud container clusters create ha-cluster \
  --region=us-central1 \
  --num-nodes=1 \
  --node-locations=us-central1-a,us-central1-b,us-central1-c

# Create autopilot cluster (fully managed)
gcloud container clusters create-auto autopilot-cluster \
  --region=us-central1

# Create cluster with autoscaling
gcloud container clusters create autoscale-cluster \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10 \
  --num-nodes=3

# Create cluster with specific Kubernetes version
gcloud container clusters create versioned-cluster \
  --zone=us-central1-a \
  --cluster-version=1.27 \
  --num-nodes=3

# Create private cluster
gcloud container clusters create private-cluster \
  --zone=us-central1-a \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --num-nodes=3

# Get cluster credentials
gcloud container clusters get-credentials my-cluster \
  --zone=us-central1-a
```

**Managing Clusters**:
```bash
# List clusters
gcloud container clusters list

# Describe cluster
gcloud container clusters describe my-cluster \
  --zone=us-central1-a

# Resize cluster
gcloud container clusters resize my-cluster \
  --num-nodes=5 \
  --zone=us-central1-a

# Upgrade cluster
gcloud container clusters upgrade my-cluster \
  --zone=us-central1-a \
  --master \
  --cluster-version=1.27

# Upgrade node pools
gcloud container clusters upgrade my-cluster \
  --zone=us-central1-a \
  --node-pool=default-pool

# Delete cluster
gcloud container clusters delete my-cluster \
  --zone=us-central1-a
```

### Node Pools

**Creating and Managing Node Pools**:
```bash
# Create node pool
gcloud container node-pools create high-mem-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --machine-type=n1-highmem-4 \
  --num-nodes=2

# Create node pool with autoscaling
gcloud container node-pools create autoscale-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5 \
  --num-nodes=2

# Create preemptible node pool
gcloud container node-pools create preemptible-pool \
  --cluster=my-cluster \
  --zone=us-central1-a \
  --preemptible \
  --num-nodes=3

# List node pools
gcloud container node-pools list \
  --cluster=my-cluster \
  --zone=us-central1-a

# Delete node pool
gcloud container node-pools delete high-mem-pool \
  --cluster=my-cluster \
  --zone=us-central1-a
```

### Workload Deployment

**Deployment YAML Example**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/my-project/nginx:v1
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Deployment Commands**:
```bash
# Create deployment
kubectl create deployment nginx --image=gcr.io/my-project/nginx:v1

# Create deployment with replicas
kubectl create deployment web --image=nginx --replicas=3

# Apply deployment from YAML
kubectl apply -f deployment.yaml

# Update deployment image
kubectl set image deployment/nginx nginx=gcr.io/my-project/nginx:v2

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Check deployment status
kubectl rollout status deployment/nginx

# View deployment history
kubectl rollout history deployment/nginx

# Rollback deployment
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx
```

### Services

**Service YAML Examples**:
```yaml
# ClusterIP Service (internal only)
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---
# NodePort Service (external access via node IP)
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080

---
# LoadBalancer Service (external load balancer)
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Service Commands**:
```bash
# Create ClusterIP service
kubectl expose deployment nginx --port=80 --type=ClusterIP

# Create LoadBalancer service
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Create service from YAML
kubectl apply -f service.yaml

# List services
kubectl get services

# Describe service
kubectl describe service nginx-service

# Delete service
kubectl delete service nginx-service
```

### Ingress

**Ingress YAML Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
    kubernetes.io/ingress.global-static-ip-name: "web-static-ip"
    networking.gke.io/managed-certificates: "web-ssl-cert"
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 3000
```

**Managed Certificate**:
```yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: web-ssl-cert
spec:
  domains:
    - example.com
    - app.example.com
```

### ConfigMaps and Secrets

**ConfigMap YAML**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db.example.com:5432"
  log_level: "INFO"
  feature_flags: |
    {
      "new_ui": true,
      "beta_features": false
    }
```

**ConfigMap Commands**:
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config \
  --from-literal=database_url=postgres://db.example.com:5432 \
  --from-literal=log_level=INFO

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties

# Create ConfigMap from directory
kubectl create configmap app-config --from-file=config-dir/

# Apply ConfigMap from YAML
kubectl apply -f configmap.yaml

# View ConfigMap
kubectl get configmap app-config -o yaml
```

**Using ConfigMap in Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: gcr.io/my-project/app:v1
        envFrom:
        - configMapRef:
            name: app-config
        # Or mount as volume
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

**Secret YAML**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQxMjM=  # base64 encoded
```

**Secret Commands**:
```bash
# Create secret from literal values
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=password123

# Create secret from file
kubectl create secret generic ssh-key \
  --from-file=id_rsa=~/.ssh/id_rsa

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# Apply secret from YAML
kubectl apply -f secret.yaml

# View secret (values are base64 encoded)
kubectl get secret db-credentials -o yaml

# Decode secret value
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d
```

**Using Secrets in Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: gcr.io/my-project/app:v1
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        # Or mount as volume
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: db-credentials
```

### GKE Autoscaling

**Horizontal Pod Autoscaler (HPA)**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**HPA Commands**:
```bash
# Create HPA based on CPU
kubectl autoscale deployment nginx --cpu-percent=70 --min=2 --max=10

# Apply HPA from YAML
kubectl apply -f hpa.yaml

# View HPA status
kubectl get hpa

# Describe HPA
kubectl describe hpa nginx-hpa
```

**Cluster Autoscaler**:
```bash
# Enable cluster autoscaler
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10 \
  --zone=us-central1-a

# Update autoscaling limits
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=3 \
  --max-nodes=15 \
  --zone=us-central1-a
```

**Vertical Pod Autoscaler (VPA)**:
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  updatePolicy:
    updateMode: "Auto"
```

## App Engine

### Environments Comparison

| Feature | Standard Environment | Flexible Environment |
|---------|---------------------|---------------------|
| **Startup Time** | Milliseconds | Minutes |
| **Scaling** | Instant, scales to zero | Gradual, minimum 1 instance |
| **Instance Shutdown** | After request | Can run background threads |
| **Pricing** | Per instance hour | Per vCPU/memory hour |
| **SSH Access** | No | Yes |
| **Custom Runtime** | No | Yes (Docker) |
| **Write to Disk** | /tmp only | Yes, ephemeral |
| **Third-party Binaries** | Limited | Yes |
| **Network Access** | Via URL Fetch API | Unrestricted |
| **Location** | Regional | Regional |
| **Free Tier** | Yes | No |
| **Max Request Timeout** | 1-10 minutes | 60 minutes |

**Standard Environment**:
- Sandboxed runtime
- Rapid scaling (milliseconds)
- Free tier available
- Language runtimes: Python, Java, Node.js, PHP, Ruby, Go
- Restrictions on system access and libraries
- Best for: Web applications with variable traffic

**Flexible Environment**:
- Docker containers
- Gradual scaling (minutes)
- More flexibility, fewer restrictions
- Custom runtimes supported
- SSH access for debugging
- No free tier
- Best for: Containerized apps, custom runtimes, background workers

### Key Features
- Automatic scaling
- Traffic splitting
- Versioning
- Built-in security
- Integrated monitoring
- Custom domains and SSL
- Memcache and task queues

### app.yaml Configuration

**Standard Environment Example**:
```yaml
runtime: python39
instance_class: F2

# Automatic scaling
automatic_scaling:
  min_instances: 1
  max_instances: 10
  min_idle_instances: 1
  max_idle_instances: 2
  target_cpu_utilization: 0.6
  target_throughput_utilization: 0.6
  max_concurrent_requests: 80

# Environment variables
env_variables:
  DATABASE_URL: "postgres://db.example.com:5432"
  CACHE_ENABLED: "true"

# Handlers
handlers:
- url: /static
  static_dir: static
  secure: always

- url: /.*
  script: auto
  secure: always

# VPC connector (for VPC access)
vpc_access_connector:
  name: "projects/PROJECT/locations/REGION/connectors/CONNECTOR"
```

**Basic Scaling Configuration**:
```yaml
runtime: python39
instance_class: F2

basic_scaling:
  max_instances: 5
  idle_timeout: 10m

env_variables:
  APP_ENV: "production"
```

**Manual Scaling Configuration**:
```yaml
runtime: python39
instance_class: F2

manual_scaling:
  instances: 3

env_variables:
  APP_ENV: "production"
```

**Flexible Environment Example**:
```yaml
runtime: python
env: flex
entrypoint: gunicorn -b :$PORT main:app

# Automatic scaling
automatic_scaling:
  min_num_instances: 2
  max_num_instances: 10
  cool_down_period_sec: 120
  cpu_utilization:
    target_utilization: 0.6

# Resources
resources:
  cpu: 2
  memory_gb: 4
  disk_size_gb: 10

# Environment variables
env_variables:
  DATABASE_URL: "postgres://db.example.com:5432"

# Network
network:
  forwarded_ports:
    - 8080
  instance_tag: app-server
  name: default

# VPC connector
vpc_access_connector:
  name: "projects/PROJECT/locations/REGION/connectors/CONNECTOR"
```

**Custom Runtime (Flexible)**:
```yaml
runtime: custom
env: flex

# Dockerfile will be used for building
```

### Deployment Commands

```bash
# Deploy app (creates new version)
gcloud app deploy

# Deploy with specific version name
gcloud app deploy --version=v1

# Deploy without promoting to default
gcloud app deploy --no-promote

# Deploy to specific service
gcloud app deploy app.yaml --service=backend

# Deploy multiple services
gcloud app deploy frontend.yaml backend.yaml worker.yaml

# View app in browser
gcloud app browse

# Stream logs
gcloud app logs tail -s default

# View app details
gcloud app describe

# View app versions
gcloud app versions list

# Stop a version
gcloud app versions stop v1

# Start a version
gcloud app versions start v1

# Delete a version
gcloud app versions delete v1
```

### Versions and Traffic Splitting

**Managing Versions**:
```bash
# List all versions
gcloud app versions list

# List versions for specific service
gcloud app versions list --service=backend

# Describe version
gcloud app versions describe v1 --service=default

# Delete old versions
gcloud app versions delete v1 v2 v3 --service=default
```

**Traffic Splitting**:
```bash
# Split traffic between versions (IP-based)
gcloud app services set-traffic default \
  --splits v1=0.9,v2=0.1

# Split traffic with cookie-based splitting (gradual rollout)
gcloud app services set-traffic default \
  --splits v1=0.8,v2=0.2 \
  --split-by=cookie

# Split traffic randomly
gcloud app services set-traffic default \
  --splits v1=0.5,v2=0.5 \
  --split-by=random

# Migrate all traffic to new version
gcloud app services set-traffic default \
  --splits v2=1

# Gradual migration example
# Day 1: 10% to v2
gcloud app services set-traffic default --splits v1=0.9,v2=0.1
# Day 2: 50% to v2
gcloud app services set-traffic default --splits v1=0.5,v2=0.5
# Day 3: 100% to v2
gcloud app services set-traffic default --splits v2=1
```

**Traffic Splitting Methods**:
- **IP**: Split based on IP address (default)
- **Cookie**: Split based on cookie (sticky sessions)
- **Random**: Random distribution (better for testing)

### Services (Microservices)

**Multiple Services Configuration**:
```bash
# Directory structure
my-app/
  app.yaml          # Default service
  frontend.yaml     # Frontend service
  backend.yaml      # Backend service
  worker.yaml       # Worker service
```

**frontend.yaml**:
```yaml
runtime: python39
service: frontend

instance_class: F2

automatic_scaling:
  min_instances: 2
  max_instances: 10

handlers:
- url: /static
  static_dir: static

- url: /.*
  script: auto
```

**backend.yaml**:
```yaml
runtime: python39
service: backend

instance_class: F4

automatic_scaling:
  min_instances: 1
  max_instances: 5

handlers:
- url: /.*
  script: auto
  login: required  # Require authentication
```

**Accessing Services**:
- Default service: `https://PROJECT_ID.REGION_ID.r.appspot.com`
- Named service: `https://SERVICE-dot-PROJECT_ID.REGION_ID.r.appspot.com`
- Specific version: `https://VERSION-dot-SERVICE-dot-PROJECT_ID.REGION_ID.r.appspot.com`

## Cloud Run

### Overview
- Fully managed serverless platform for containers
- Automatic scaling (including to zero)
- Pay only for resources used
- Built on Knative
- Supports any language/library/binary

### Deployment

**Deploy from Container Image**:
```bash
# Deploy from GCR
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --platform=managed \
  --region=us-central1 \
  --allow-unauthenticated

# Deploy from Artifact Registry
gcloud run deploy my-service \
  --image=us-central1-docker.pkg.dev/my-project/my-repo/my-image:v1 \
  --region=us-central1

# Deploy with specific memory and CPU
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --memory=1Gi \
  --cpu=2 \
  --region=us-central1

# Deploy with environment variables
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --set-env-vars="DATABASE_URL=postgres://...,API_KEY=abc123" \
  --region=us-central1

# Deploy with secrets from Secret Manager
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --set-secrets="DB_PASSWORD=db-password:latest" \
  --region=us-central1

# Deploy with concurrency limit
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --concurrency=80 \
  --region=us-central1

# Deploy with custom service account
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v1 \
  --service-account=my-sa@my-project.iam.gserviceaccount.com \
  --region=us-central1
```

**Deploy from Source (Cloud Build)**:
```bash
# Deploy directly from source code
gcloud run deploy my-service \
  --source=. \
  --region=us-central1

# This will:
# 1. Build container using buildpacks
# 2. Push to Artifact Registry
# 3. Deploy to Cloud Run
```

### Traffic Management

**Traffic Splitting and Revisions**:
```bash
# List revisions
gcloud run revisions list --service=my-service --region=us-central1

# Deploy new revision without sending traffic
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image:v2 \
  --no-traffic \
  --region=us-central1

# Split traffic between revisions
gcloud run services update-traffic my-service \
  --to-revisions=my-service-001=90,my-service-002=10 \
  --region=us-central1

# Send all traffic to specific revision
gcloud run services update-traffic my-service \
  --to-revisions=my-service-002=100 \
  --region=us-central1

# Tag a revision
gcloud run services update-traffic my-service \
  --update-tags=staging=my-service-002 \
  --region=us-central1

# Access tagged revision
# https://staging---my-service-hash-uc.a.run.app
```

**Revision Management**:
```bash
# Describe revision
gcloud run revisions describe my-service-001 \
  --region=us-central1

# Delete revision
gcloud run revisions delete my-service-001 \
  --region=us-central1

# List revisions with traffic split
gcloud run services describe my-service \
  --region=us-central1 \
  --format="value(status.traffic)"
```

### Cloud Run YAML Configuration

**service.yaml Example**:
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-service
  annotations:
    run.googleapis.com/ingress: all
    run.googleapis.com/ingress-status: all
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "10"
        run.googleapis.com/vpc-access-connector: projects/PROJECT/locations/REGION/connectors/CONNECTOR
    spec:
      containerConcurrency: 80
      timeoutSeconds: 300
      serviceAccountName: my-sa@my-project.iam.gserviceaccount.com
      containers:
      - image: gcr.io/my-project/my-image:v1
        ports:
        - name: http1
          containerPort: 8080
        env:
        - name: DATABASE_URL
          value: postgres://db.example.com:5432
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-key
              key: latest
        resources:
          limits:
            memory: 1Gi
            cpu: "2"
```

**Deploy from YAML**:
```bash
gcloud run services replace service.yaml --region=us-central1
```

### Autoscaling Configuration

```bash
# Set minimum and maximum instances
gcloud run services update my-service \
  --min-instances=1 \
  --max-instances=100 \
  --region=us-central1

# Set concurrency (requests per container)
gcloud run services update my-service \
  --concurrency=80 \
  --region=us-central1

# Set CPU throttling (always allocated vs only during request)
gcloud run services update my-service \
  --cpu-throttling \
  --region=us-central1

# Always allocate CPU (for background tasks)
gcloud run services update my-service \
  --no-cpu-throttling \
  --region=us-central1
```

### Networking and Security

```bash
# Update IAM policy (public access)
gcloud run services add-iam-policy-binding my-service \
  --member="allUsers" \
  --role="roles/run.invoker" \
  --region=us-central1

# Update IAM policy (authenticated access)
gcloud run services add-iam-policy-binding my-service \
  --member="allAuthenticatedUsers" \
  --role="roles/run.invoker" \
  --region=us-central1

# Grant access to specific service account
gcloud run services add-iam-policy-binding my-service \
  --member="serviceAccount:caller@project.iam.gserviceaccount.com" \
  --role="roles/run.invoker" \
  --region=us-central1

# Connect to VPC
gcloud run services update my-service \
  --vpc-connector=my-connector \
  --region=us-central1

# Route all traffic through VPC
gcloud run services update my-service \
  --vpc-egress=all-traffic \
  --region=us-central1

# Route only private traffic through VPC
gcloud run services update my-service \
  --vpc-egress=private-ranges-only \
  --region=us-central1

# Set ingress settings
gcloud run services update my-service \
  --ingress=internal \
  --region=us-central1
# Options: all (default), internal, internal-and-cloud-load-balancing
```

### Managing Services

```bash
# List services
gcloud run services list --platform=managed

# Describe service
gcloud run services describe my-service \
  --region=us-central1

# Update service
gcloud run services update my-service \
  --image=gcr.io/my-project/my-image:v2 \
  --region=us-central1

# Delete service
gcloud run services delete my-service \
  --region=us-central1

# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=my-service" \
  --limit=50 \
  --format=json
```

## Cloud Functions

### Overview
- Event-driven serverless functions
- Pay per invocation and duration
- Auto-scaling
- No server management
- Simpler than Cloud Run for event-driven tasks

### Generations Comparison

| Feature | 1st Gen | 2nd Gen (Recommended) |
|---------|---------|----------------------|
| **Runtime** | Cloud Functions | Cloud Run |
| **Max Timeout** | 9 minutes | 60 minutes |
| **Max Memory** | 8GB | 32GB |
| **Max Instances** | 3000 | 1000 |
| **Concurrency** | 1 | Up to 1000 |
| **Min Instances** | 0 | 0 |
| **Event Types** | Limited | More options |
| **Deployment** | Function only | Container-based |

**1st gen**: Original Cloud Functions
**2nd gen**: Built on Cloud Run, more features and flexibility

### Trigger Types
- **HTTP**: HTTP(S) requests
- **Cloud Storage**: Object create, delete, archive, metadata update
- **Pub/Sub**: Message published to topic
- **Firestore**: Document create, update, delete
- **Firebase**: Authentication, Realtime Database, Analytics
- **Cloud Scheduler**: Scheduled execution

### Runtime Environments
- Node.js (16, 18, 20)
- Python (3.8, 3.9, 3.10, 3.11)
- Go (1.16, 1.18, 1.19, 1.20, 1.21)
- Java (11, 17)
- .NET (3.1, 6)
- Ruby (2.7, 3.0, 3.1, 3.2)
- PHP (7.4, 8.1, 8.2)

### Function Deployment

**HTTP Triggered Function**:
```bash
# Deploy HTTP function (2nd gen)
gcloud functions deploy hello-http \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=hello_http \
  --trigger-http \
  --allow-unauthenticated

# Deploy with environment variables
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --set-env-vars="DATABASE_URL=postgres://...,API_KEY=abc123"

# Deploy with secrets
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --set-secrets="DB_PASSWORD=db-password:latest"

# Deploy with memory and timeout
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --memory=512MB \
  --timeout=120s

# Deploy with specific service account
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --service-account=my-sa@my-project.iam.gserviceaccount.com
```

**Cloud Storage Triggered Function**:
```bash
# Trigger on object create
gcloud functions deploy process-upload \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_file \
  --trigger-bucket=my-bucket

# Trigger on specific event type
gcloud functions deploy handle-delete \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=handle_delete \
  --trigger-event-filters="type=google.cloud.storage.object.v1.deleted" \
  --trigger-event-filters="bucket=my-bucket"
```

**Pub/Sub Triggered Function**:
```bash
# Trigger on Pub/Sub message
gcloud functions deploy process-message \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_pubsub \
  --trigger-topic=my-topic

# Create topic first if needed
gcloud pubsub topics create my-topic

# Test by publishing message
gcloud pubsub topics publish my-topic --message="test message"
```

**Scheduled Function (Cloud Scheduler)**:
```bash
# Deploy HTTP function
gcloud functions deploy scheduled-task \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=scheduled_task \
  --trigger-http \
  --no-allow-unauthenticated

# Create Cloud Scheduler job
gcloud scheduler jobs create http scheduled-job \
  --schedule="0 2 * * *" \
  --uri="https://us-central1-my-project.cloudfunctions.net/scheduled-task" \
  --http-method=POST \
  --oidc-service-account-email=scheduler@my-project.iam.gserviceaccount.com
```

### Function Code Examples

**Python HTTP Function**:
```python
# main.py
import functions_framework
import os

@functions_framework.http
def hello_http(request):
    name = request.args.get('name', 'World')
    database_url = os.environ.get('DATABASE_URL')
    return f'Hello {name}!'
```

**Python Storage Function**:
```python
# main.py
import functions_framework
from google.cloud import storage

@functions_framework.cloud_event
def process_file(cloud_event):
    data = cloud_event.data
    bucket = data["bucket"]
    name = data["name"]

    print(f"Processing file: gs://{bucket}/{name}")
    # Process file logic here
```

**Python Pub/Sub Function**:
```python
# main.py
import functions_framework
import base64
import json

@functions_framework.cloud_event
def process_pubsub(cloud_event):
    # Decode Pub/Sub message
    message = base64.b64decode(cloud_event.data["message"]["data"]).decode()
    print(f"Received message: {message}")
    # Process message logic here
```

### Configuration

**Memory and CPU Options**:
- Memory: 128MB, 256MB, 512MB, 1GB, 2GB, 4GB, 8GB (1st gen)
- Memory: 128MB to 32GB (2nd gen)
- CPU: Automatically scaled with memory

**Timeout**:
- 1st gen: Max 9 minutes (540 seconds)
- 2nd gen: Max 60 minutes (3600 seconds)

**Concurrency**:
- 1st gen: 1 request per instance
- 2nd gen: Up to 1000 concurrent requests per instance

### Networking

```bash
# Connect function to VPC
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --vpc-connector=projects/PROJECT/locations/REGION/connectors/CONNECTOR

# Set egress settings
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --vpc-connector=my-connector \
  --egress-settings=all
# Options: private-ranges-only, all

# Set ingress settings
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --ingress-settings=internal-only
# Options: all, internal-only, internal-and-gclb
```

### Managing Functions

```bash
# List functions
gcloud functions list --gen2 --region=us-central1

# Describe function
gcloud functions describe my-function \
  --gen2 \
  --region=us-central1

# View logs
gcloud functions logs read my-function \
  --gen2 \
  --region=us-central1 \
  --limit=50

# Delete function
gcloud functions delete my-function \
  --gen2 \
  --region=us-central1

# Call HTTP function
gcloud functions call my-function \
  --gen2 \
  --region=us-central1 \
  --data='{"name":"test"}'

# Update function
gcloud functions deploy my-function \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=main \
  --trigger-http \
  --update-env-vars="NEW_VAR=value"
```

## Troubleshooting Scenarios

### Scenario 1: GKE Pod Not Starting

**Problem**: Deployed application to GKE, but pods remain in Pending state.

**Diagnosis Steps**:
1. Check pod status and events:
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

2. Look for common issues in events:
   - Insufficient CPU/memory resources
   - Image pull errors
   - Node selector mismatch
   - PersistentVolumeClaim not bound

3. Check node capacity:
```bash
kubectl get nodes
kubectl describe node <node-name>
```

4. Check for resource quotas:
```bash
kubectl get resourcequotas
```

**Common Solutions**:
- **Insufficient resources**: Increase node pool size or enable cluster autoscaling
```bash
gcloud container clusters update my-cluster \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10 \
  --zone=us-central1-a
```

- **Image pull error**: Verify image exists and service account has permissions
```bash
# Grant permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_EMAIL \
  --role=roles/storage.objectViewer
```

- **PVC not bound**: Check if storage class exists and has available storage
```bash
kubectl get pvc
kubectl get storageclass
```

### Scenario 2: Cloud Run Service Returning 5xx Errors

**Problem**: Cloud Run service deployed successfully but returns 500 errors.

**Diagnosis Steps**:
1. Check service logs:
```bash
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=my-service" \
  --limit=50 \
  --format=json
```

2. Verify container is listening on correct port:
   - Cloud Run requires listening on $PORT environment variable
   - Default is 8080

3. Check startup time and timeout settings:
```bash
gcloud run services describe my-service \
  --region=us-central1 \
  --format="value(spec.template.spec.timeoutSeconds)"
```

4. Test container locally:
```bash
docker run -p 8080:8080 -e PORT=8080 gcr.io/my-project/my-image:v1
curl http://localhost:8080
```

**Common Solutions**:
- **Port mismatch**: Update application to use $PORT
```python
# Python example
import os
port = int(os.environ.get('PORT', 8080))
app.run(host='0.0.0.0', port=port)
```

- **Timeout too short**: Increase timeout
```bash
gcloud run services update my-service \
  --timeout=300 \
  --region=us-central1
```

- **Cold start issues**: Set minimum instances
```bash
gcloud run services update my-service \
  --min-instances=1 \
  --region=us-central1
```

- **Memory issues**: Increase memory limit
```bash
gcloud run services update my-service \
  --memory=2Gi \
  --region=us-central1
```

### Scenario 3: App Engine Deployment Fails

**Problem**: `gcloud app deploy` fails with error.

**Diagnosis Steps**:
1. Check deployment output for specific error
2. Verify app.yaml syntax:
```bash
# Common syntax issues:
# - Incorrect indentation
# - Missing required fields
# - Invalid runtime version
```

3. Check App Engine quotas:
```bash
gcloud app describe
```

4. Verify service account permissions
5. Check if region is enabled for App Engine

**Common Solutions**:
- **Region not initialized**: Initialize App Engine
```bash
gcloud app create --region=us-central
```

- **Invalid runtime**: Use supported runtime version
```yaml
# Correct
runtime: python311

# Incorrect
runtime: python3.11
```

- **File size too large**: Use .gcloudignore to exclude files
```bash
# .gcloudignore
node_modules/
.git/
*.pyc
```

- **Quota exceeded**: Request quota increase or clean up old versions
```bash
# Delete old versions
gcloud app versions list
gcloud app versions delete v1 v2 v3
```

### Scenario 4: MIG Rolling Update Stuck

**Problem**: MIG rolling update not completing, instances stuck in unhealthy state.

**Diagnosis Steps**:
1. Check update status:
```bash
gcloud compute instance-groups managed describe web-mig \
  --zone=us-central1-a
```

2. Check health check status:
```bash
gcloud compute health-checks describe web-health-check
```

3. Verify instance startup:
```bash
gcloud compute instances get-serial-port-output INSTANCE_NAME \
  --zone=us-central1-a
```

4. Check application logs on instance:
```bash
gcloud compute ssh INSTANCE_NAME --zone=us-central1-a
sudo journalctl -u application-service
```

**Common Solutions**:
- **Health check failing**: Adjust health check parameters
```bash
gcloud compute health-checks update http web-health-check \
  --check-interval=30s \
  --timeout=10s \
  --unhealthy-threshold=5 \
  --healthy-threshold=2
```

- **Initial delay too short**: Increase autohealing initial delay
```bash
gcloud compute instance-groups managed set-autohealing web-mig \
  --health-check=web-health-check \
  --initial-delay=600 \
  --zone=us-central1-a
```

- **Startup script failing**: Test startup script manually
```bash
gcloud compute ssh INSTANCE_NAME --zone=us-central1-a
# Run startup script commands manually
```

- **Cancel bad update**: Stop the update
```bash
gcloud compute instance-groups managed rolling-action stop-proactive-update web-mig \
  --zone=us-central1-a
```

### Scenario 5: Cloud Function Timeouts

**Problem**: Cloud Function executes but times out before completion.

**Diagnosis Steps**:
1. Check function logs:
```bash
gcloud functions logs read my-function \
  --gen2 \
  --region=us-central1 \
  --limit=50
```

2. Check function timeout setting:
```bash
gcloud functions describe my-function \
  --gen2 \
  --region=us-central1
```

3. Identify slow operations:
   - Database queries
   - External API calls
   - File processing

**Common Solutions**:
- **Increase timeout**: Update function configuration
```bash
gcloud functions deploy my-function \
  --gen2 \
  --timeout=540s \
  --region=us-central1
```

- **Optimize code**: Use asynchronous operations
```python
# Instead of sequential
result1 = api_call_1()
result2 = api_call_2()

# Use concurrent
import asyncio
results = await asyncio.gather(
    api_call_1(),
    api_call_2()
)
```

- **Move to Cloud Run**: If need more than 9 minutes (1st gen)
```bash
# Cloud Run supports up to 60 minutes timeout
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-image \
  --timeout=3600 \
  --region=us-central1
```

- **Use Cloud Tasks**: For longer operations, enqueue tasks
```bash
gcloud tasks create-app-engine-task \
  --queue=my-queue \
  --method=POST \
  --url=/process \
  --schedule-time=2024-01-15T12:00:00Z
```

### Scenario 6: GKE Ingress SSL Certificate Not Working

**Problem**: Ingress deployed but HTTPS not working, certificate not provisioned.

**Diagnosis Steps**:
1. Check Ingress status:
```bash
kubectl describe ingress web-ingress
```

2. Check ManagedCertificate status:
```bash
kubectl describe managedcertificate web-ssl-cert
```

3. Verify DNS configuration:
```bash
nslookup example.com
# Should point to Ingress IP
```

4. Check certificate events:
```bash
kubectl get events --field-selector involvedObject.name=web-ssl-cert
```

**Common Solutions**:
- **DNS not pointing to Ingress**: Update DNS records
```bash
# Get Ingress IP
kubectl get ingress web-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Create A record pointing domain to this IP
```

- **Certificate pending**: Wait for provisioning (can take 15-60 minutes)
```bash
# Check status
kubectl get managedcertificate web-ssl-cert -o yaml
# Status should show "Active"
```

- **Domain validation failing**: Ensure DNS is correctly configured
```yaml
# ManagedCertificate must match Ingress host
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: web-ssl-cert
spec:
  domains:
    - example.com    # Must match Ingress host
```

- **Using reserved IP**: Reserve and use static IP
```bash
# Reserve IP
gcloud compute addresses create web-static-ip --global

# Update Ingress
kubectl annotate ingress web-ingress \
  kubernetes.io/ingress.global-static-ip-name=web-static-ip
```

### Scenario 7: App Engine Traffic Splitting Not Working

**Problem**: Traffic splitting configured but all traffic goes to one version.

**Diagnosis Steps**:
1. Check current traffic split:
```bash
gcloud app services describe default
```

2. Verify versions are running:
```bash
gcloud app versions list
```

3. Check split configuration:
```bash
gcloud app services describe default --format="value(split)"
```

**Common Solutions**:
- **Versions not serving**: Start stopped versions
```bash
gcloud app versions start v2
```

- **Split percentages don't add to 1.0**: Fix split values
```bash
# Correct split (must sum to 1.0)
gcloud app services set-traffic default \
  --splits v1=0.8,v2=0.2

# Or use whole numbers (must sum to 100)
gcloud app services set-traffic default \
  --splits v1=80,v2=20
```

- **IP-based splitting with same IP**: Use cookie-based splitting
```bash
gcloud app services set-traffic default \
  --splits v1=0.8,v2=0.2 \
  --split-by=cookie
```

- **Migration needed**: Explicitly migrate traffic
```bash
gcloud app services set-traffic default \
  --splits v2=1 \
  --migrate
```

### Scenario 8: Compute Engine Instance Template Update Not Applied

**Problem**: Updated instance template but MIG instances still use old configuration.

**Diagnosis Steps**:
1. Verify template was updated:
```bash
gcloud compute instance-templates describe new-template
```

2. Check MIG configuration:
```bash
gcloud compute instance-groups managed describe web-mig \
  --zone=us-central1-a
```

3. Check if rolling update was started:
```bash
gcloud compute instance-groups managed describe web-mig \
  --zone=us-central1-a \
  --format="value(status)"
```

**Common Solutions**:
- **Forgot to start rolling update**: Initiate update
```bash
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=new-template \
  --zone=us-central1-a
```

- **Update with wrong parameters**: Set correct surge/unavailable
```bash
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=new-template \
  --max-surge=3 \
  --max-unavailable=0 \
  --zone=us-central1-a
```

- **Need immediate update**: Use replace action
```bash
gcloud compute instance-groups managed rolling-action replace web-mig \
  --version=template=new-template \
  --zone=us-central1-a
```

- **Update MIG template**: Set new template as default
```bash
gcloud compute instance-groups managed set-instance-template web-mig \
  --template=new-template \
  --zone=us-central1-a
```

## Service Selection Guide

### Compute Service Comparison

| Service | Best For | Scaling | Management | Cold Start | Cost Model |
|---------|----------|---------|------------|------------|------------|
| **Compute Engine** | Full VM control, legacy apps | Manual/Auto (MIGs) | Self-managed | Minutes | Per-second |
| **GKE** | Microservices, containers | Auto (HPA/VPA/CA) | Managed Kubernetes | Seconds | Per-second |
| **App Engine Standard** | Web apps, variable traffic | Instant to zero | Fully managed | Milliseconds | Instance-hour |
| **App Engine Flexible** | Containers, custom runtimes | Minutes, min 1 | Fully managed | Minutes | vCPU/memory-hour |
| **Cloud Run** | Stateless containers | Instant to zero | Fully managed | Milliseconds | Per-request |
| **Cloud Functions** | Event-driven, short tasks | Instant to zero | Fully managed | Milliseconds | Per-invocation |

### Decision Tree

**Need Kubernetes?**
- Yes → **GKE**
- No → Continue

**Need VMs?**
- Yes → **Compute Engine** + MIGs
- No → Continue

**Event-driven workload?**
- Yes → **Cloud Functions** (simple) or **Cloud Run** (complex)
- No → Continue

**Existing App Engine app?**
- Yes → **App Engine**
- No → Continue

**Containerized application?**
- Yes → **Cloud Run** (recommended) or **App Engine Flexible**
- No → **App Engine Standard**

### Common Use Cases

**Compute Engine**:
- Legacy applications requiring specific OS
- Windows workloads
- GPU-intensive workloads
- Full control over infrastructure
- Stateful applications

**GKE**:
- Microservices architecture
- Multi-cloud Kubernetes workloads
- Complex orchestration requirements
- Need for service mesh
- Batch processing jobs

**App Engine Standard**:
- Web applications with variable traffic
- Rapid prototyping
- Cost-sensitive projects (free tier)
- Python/Java/Node.js/Go web apps

**App Engine Flexible**:
- Custom runtimes
- Applications requiring SSH access
- Gradual scaling requirements
- Docker-based applications

**Cloud Run**:
- Stateless HTTP services
- API backends
- Webhook handlers
- Microservices
- Any containerized application

**Cloud Functions**:
- Image/video processing (Storage triggers)
- Real-time file processing
- Lightweight APIs
- Scheduled tasks
- IoT data processing (Pub/Sub)
- Webhooks and integrations

## Exam Tips

### Critical Service Selection Rules

1. **Serverless First**: For new projects without specific VM needs, choose serverless (Cloud Run, Functions, App Engine)

2. **Container vs Code**:
   - Have container? → Cloud Run
   - Have source code? → App Engine Standard or Cloud Functions

3. **Event-Driven**:
   - Simple event handler (< 15 minutes)? → Cloud Functions
   - Complex event handler? → Cloud Run

4. **Scaling Requirements**:
   - Scale to zero needed? → Cloud Run, Cloud Functions, App Engine Standard
   - Minimum instances required? → App Engine Flexible, GKE
   - Full control? → Compute Engine MIGs

5. **Kubernetes**:
   - Need Kubernetes API? → GKE
   - Don't need Kubernetes? → Don't use GKE (use simpler option)

### Common Deployment Traps

1. **App Engine Region Lock**:
   - App Engine region cannot be changed after creation
   - Choose region carefully during `gcloud app create`

2. **Cloud Run Port Requirements**:
   - Must listen on $PORT environment variable
   - Default port assumptions will fail

3. **MIG Template Updates**:
   - Updating template doesn't update instances
   - Must run rolling-action start-update

4. **GKE Cluster Autoscaling**:
   - Cluster autoscaler adds nodes
   - HPA scales pods
   - Need BOTH for complete autoscaling

5. **Traffic Splitting Math**:
   - App Engine splits must sum to 1.0 (or 100)
   - Cloud Run requires exact revision names

6. **Health Check Timing**:
   - Initial delay must exceed startup time
   - Too short = instances repeatedly killed
   - Too long = slow failure detection

7. **Cloud Functions Timeout**:
   - 1st gen: Max 9 minutes
   - 2nd gen: Max 60 minutes
   - For longer jobs, use Cloud Run or Compute Engine

8. **Secret vs Environment Variable**:
   - Use Secret Manager for sensitive data
   - Environment variables are visible in console
   - Secrets provide audit logging and versioning

### Key Differences to Remember

**App Engine Standard vs Flexible**:
- Standard: Faster scaling, limited runtime, free tier
- Flexible: Custom runtime, SSH access, no free tier

**Cloud Functions vs Cloud Run**:
- Functions: Event-driven, simpler, per-invocation billing
- Run: HTTP-focused, containers, per-request billing

**GKE Standard vs Autopilot**:
- Standard: Full control, node management
- Autopilot: Fully managed, pay-per-pod

**Regional vs Zonal**:
- Regional: Higher availability, higher cost (MIG, GKE)
- Zonal: Lower cost, single zone failure risk

### Exam Command Gotchas

1. **Always specify region/zone** (no defaults):
```bash
# Wrong
gcloud run deploy my-service --image=...

# Correct
gcloud run deploy my-service --image=... --region=us-central1
```

2. **GKE requires get-credentials before kubectl**:
```bash
gcloud container clusters get-credentials my-cluster --zone=us-central1-a
kubectl get pods
```

3. **Cloud Functions gen2 flag**:
```bash
# Recommended (gen2)
gcloud functions deploy my-func --gen2 ...

# Legacy (gen1)
gcloud functions deploy my-func ...
```

4. **MIG rolling update syntax**:
```bash
# Note: --version=template= (not --template=)
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=new-template
```

5. **App Engine service vs version**:
```bash
# Set traffic by SERVICE
gcloud app services set-traffic default --splits v1=0.8,v2=0.2

# Manage individual versions
gcloud app versions list
```

### Quick Reference Commands

**Deployment**:
```bash
# Compute Engine
gcloud compute instances create <name> --zone=<zone>

# GKE
gcloud container clusters create <name> --zone=<zone>
kubectl apply -f deployment.yaml

# App Engine
gcloud app deploy

# Cloud Run
gcloud run deploy <name> --image=<image> --region=<region>

# Cloud Functions
gcloud functions deploy <name> --runtime=<runtime> --trigger-http --region=<region>
```

**Scaling**:
```bash
# MIG autoscaling
gcloud compute instance-groups managed set-autoscaling <mig> \
  --min-num-replicas=2 --max-num-replicas=10

# GKE HPA
kubectl autoscale deployment <name> --cpu-percent=70 --min=2 --max=10

# Cloud Run
gcloud run services update <name> --min-instances=1 --max-instances=100
```

**Traffic Management**:
```bash
# App Engine
gcloud app services set-traffic default --splits v1=0.9,v2=0.1

# Cloud Run
gcloud run services update-traffic <name> \
  --to-revisions=<rev1>=90,<rev2>=10
```

### Best Practices for Exam

1. **Read requirements carefully**:
   - "Serverless" → Cloud Run, Functions, or App Engine
   - "Kubernetes" → GKE only
   - "Event-driven" → Cloud Functions preferred
   - "Scale to zero" → Cloud Run, Functions, App Engine Standard

2. **Consider cost**:
   - Mention free tier if applicable (App Engine Standard, Cloud Functions)
   - Preemptible VMs for cost savings
   - Sustained use discounts for Compute Engine

3. **Consider availability**:
   - Regional > Zonal
   - Multi-region for critical workloads
   - Health checks always recommended

4. **Security defaults**:
   - Least privilege IAM
   - Service accounts for workload identity
   - Secret Manager for credentials
   - VPC for internal communication

5. **Monitoring**:
   - Enable Cloud Logging and Monitoring
   - Set up alerts for production
   - Use health checks
