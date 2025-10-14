# Google Cloud Platform (GCP) CLI Cheat Sheet

A comprehensive reference for gcloud and gsutil commands covering essential GCP services for cloud certification and daily operations.

## Table of Contents
- [Configuration & Setup](#configuration--setup)
- [Compute Engine](#compute-engine)
- [Cloud Storage (gsutil)](#cloud-storage-gsutil)
- [GKE (Google Kubernetes Engine)](#gke-google-kubernetes-engine)
- [Cloud SQL](#cloud-sql)
- [IAM (Identity and Access Management)](#iam-identity-and-access-management)
- [VPC (Virtual Private Cloud)](#vpc-virtual-private-cloud)
- [Cloud Functions](#cloud-functions)
- [App Engine](#app-engine)
- [BigQuery](#bigquery)
- [Cloud Build](#cloud-build)
- [Additional Services](#additional-services)

---

## Configuration & Setup

### Initialize and Configure gcloud
```bash
# Initialize gcloud
gcloud init

# Login to GCP
gcloud auth login

# Login for application default credentials
gcloud auth application-default login

# List configurations
gcloud config configurations list

# Create new configuration
gcloud config configurations create production

# Activate configuration
gcloud config configurations activate production

# Set project
gcloud config set project my-project-id

# Set default region
gcloud config set compute/region us-central1

# Set default zone
gcloud config set compute/zone us-central1-a

# View current configuration
gcloud config list

# Get specific config value
gcloud config get-value project

# Unset configuration value
gcloud config unset compute/region

# View account information
gcloud auth list

# Revoke credentials
gcloud auth revoke user@example.com

# List projects
gcloud projects list

# Describe project
gcloud projects describe my-project-id

# Get current project ID
gcloud config get-value project
```

### SDK Management
```bash
# Update gcloud components
gcloud components update

# List components
gcloud components list

# Install component
gcloud components install kubectl

# Remove component
gcloud components remove kubectl

# Check gcloud version
gcloud version

# Get help
gcloud help

# Get command help
gcloud compute instances create --help
```

---

## Compute Engine

### Instance Management
```bash
# List instances
gcloud compute instances list

# List instances in specific zone
gcloud compute instances list --zones=us-central1-a

# Describe instance
gcloud compute instances describe instance-1 --zone=us-central1-a

# Create instance
gcloud compute instances create instance-1 \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=debian-11 \
  --image-project=debian-cloud

# Create instance with startup script
gcloud compute instances create instance-1 \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata-from-file startup-script=startup.sh

# Create instance with tags
gcloud compute instances create instance-1 \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --tags=http-server,https-server

# Create preemptible instance
gcloud compute instances create instance-1 \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --preemptible

# Start instance
gcloud compute instances start instance-1 --zone=us-central1-a

# Stop instance
gcloud compute instances stop instance-1 --zone=us-central1-a

# Reset instance
gcloud compute instances reset instance-1 --zone=us-central1-a

# Delete instance
gcloud compute instances delete instance-1 --zone=us-central1-a

# Delete multiple instances
gcloud compute instances delete instance-1 instance-2 --zone=us-central1-a

# SSH into instance
gcloud compute ssh instance-1 --zone=us-central1-a

# SSH with specific user
gcloud compute ssh user@instance-1 --zone=us-central1-a

# Execute command on instance
gcloud compute ssh instance-1 --zone=us-central1-a --command="ls -la"

# Copy files to instance
gcloud compute scp local-file.txt instance-1:~/remote-file.txt --zone=us-central1-a

# Copy files from instance
gcloud compute scp instance-1:~/remote-file.txt local-file.txt --zone=us-central1-a

# Get serial port output
gcloud compute instances get-serial-port-output instance-1 --zone=us-central1-a
```

### Instance Configuration
```bash
# Change machine type (requires instance to be stopped)
gcloud compute instances set-machine-type instance-1 \
  --machine-type=e2-medium \
  --zone=us-central1-a

# Add metadata
gcloud compute instances add-metadata instance-1 \
  --zone=us-central1-a \
  --metadata=key1=value1,key2=value2

# Remove metadata
gcloud compute instances remove-metadata instance-1 \
  --zone=us-central1-a \
  --keys=key1,key2

# Add tags
gcloud compute instances add-tags instance-1 \
  --zone=us-central1-a \
  --tags=web-server,production

# Remove tags
gcloud compute instances remove-tags instance-1 \
  --zone=us-central1-a \
  --tags=web-server

# Add labels
gcloud compute instances add-labels instance-1 \
  --zone=us-central1-a \
  --labels=environment=production,team=backend

# Remove labels
gcloud compute instances remove-labels instance-1 \
  --zone=us-central1-a \
  --labels=environment,team

# Update instance
gcloud compute instances update instance-1 \
  --zone=us-central1-a \
  --deletion-protection
```

### Disks
```bash
# List disks
gcloud compute disks list

# Create disk
gcloud compute disks create disk-1 \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-standard

# Create SSD disk
gcloud compute disks create disk-1 \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-ssd

# Delete disk
gcloud compute disks delete disk-1 --zone=us-central1-a

# Attach disk to instance
gcloud compute instances attach-disk instance-1 \
  --disk=disk-1 \
  --zone=us-central1-a

# Detach disk from instance
gcloud compute instances detach-disk instance-1 \
  --disk=disk-1 \
  --zone=us-central1-a

# Resize disk
gcloud compute disks resize disk-1 \
  --size=200GB \
  --zone=us-central1-a

# Create snapshot
gcloud compute disks snapshot disk-1 \
  --snapshot-names=snapshot-1 \
  --zone=us-central1-a

# List snapshots
gcloud compute snapshots list

# Delete snapshot
gcloud compute snapshots delete snapshot-1

# Create disk from snapshot
gcloud compute disks create disk-2 \
  --source-snapshot=snapshot-1 \
  --zone=us-central1-a
```

### Images
```bash
# List images
gcloud compute images list

# List images from specific project
gcloud compute images list --project=debian-cloud

# Describe image
gcloud compute images describe image-1

# Create image from disk
gcloud compute images create image-1 \
  --source-disk=disk-1 \
  --source-disk-zone=us-central1-a

# Create image from snapshot
gcloud compute images create image-1 \
  --source-snapshot=snapshot-1

# Delete image
gcloud compute images delete image-1

# Deprecate image
gcloud compute images deprecate image-1 \
  --state=DEPRECATED \
  --replacement=image-2

# Export image to Cloud Storage
gcloud compute images export \
  --image=image-1 \
  --destination-uri=gs://my-bucket/image-1.tar.gz
```

### Instance Templates
```bash
# List instance templates
gcloud compute instance-templates list

# Create instance template
gcloud compute instance-templates create template-1 \
  --machine-type=e2-micro \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --tags=http-server

# Delete instance template
gcloud compute instance-templates delete template-1

# Describe instance template
gcloud compute instance-templates describe template-1
```

### Firewall Rules
```bash
# List firewall rules
gcloud compute firewall-rules list

# Create firewall rule (allow HTTP)
gcloud compute firewall-rules create allow-http \
  --allow=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server

# Create firewall rule (allow HTTPS)
gcloud compute firewall-rules create allow-https \
  --allow=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=https-server

# Create firewall rule (allow SSH)
gcloud compute firewall-rules create allow-ssh \
  --allow=tcp:22 \
  --source-ranges=0.0.0.0/0

# Create firewall rule with specific source
gcloud compute firewall-rules create allow-internal \
  --allow=tcp:3306 \
  --source-ranges=10.0.0.0/8

# Update firewall rule
gcloud compute firewall-rules update allow-http \
  --allow=tcp:80,tcp:8080

# Delete firewall rule
gcloud compute firewall-rules delete allow-http

# Describe firewall rule
gcloud compute firewall-rules describe allow-http
```

### Load Balancing
```bash
# Create HTTP health check
gcloud compute health-checks create http http-basic-check \
  --port=80

# Create backend service
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --health-checks=http-basic-check \
  --global

# Add backend to backend service
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=instance-group-1 \
  --instance-group-zone=us-central1-a \
  --global

# Create URL map
gcloud compute url-maps create web-map \
  --default-service=web-backend-service

# Create target HTTP proxy
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map=web-map

# Create forwarding rule
gcloud compute forwarding-rules create http-content-rule \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80

# List forwarding rules
gcloud compute forwarding-rules list
```

### Instance Groups
```bash
# Create instance group
gcloud compute instance-groups unmanaged create instance-group-1 \
  --zone=us-central1-a

# Create managed instance group
gcloud compute instance-groups managed create instance-group-1 \
  --base-instance-name=instance \
  --template=template-1 \
  --size=3 \
  --zone=us-central1-a

# List instance groups
gcloud compute instance-groups list

# Resize managed instance group
gcloud compute instance-groups managed resize instance-group-1 \
  --size=5 \
  --zone=us-central1-a

# Set autoscaling
gcloud compute instance-groups managed set-autoscaling instance-group-1 \
  --zone=us-central1-a \
  --min-num-replicas=2 \
  --max-num-replicas=10 \
  --target-cpu-utilization=0.6

# Delete instance group
gcloud compute instance-groups managed delete instance-group-1 \
  --zone=us-central1-a
```

---

## Cloud Storage (gsutil)

### Bucket Management
```bash
# List buckets
gsutil ls

# Create bucket
gsutil mb gs://my-unique-bucket-name

# Create bucket in specific location
gsutil mb -l us-central1 gs://my-unique-bucket-name

# Create bucket with storage class
gsutil mb -c nearline gs://my-unique-bucket-name

# Remove bucket
gsutil rb gs://my-unique-bucket-name

# Remove bucket with all contents
gsutil rm -r gs://my-unique-bucket-name

# Get bucket metadata
gsutil ls -L -b gs://my-unique-bucket-name

# List bucket contents
gsutil ls gs://my-bucket-name

# List recursively
gsutil ls -r gs://my-bucket-name

# List with details
gsutil ls -l gs://my-bucket-name
```

### File Operations
```bash
# Upload file
gsutil cp local-file.txt gs://my-bucket-name/

# Upload with parallel composite uploads
gsutil -o GSUtil:parallel_composite_upload_threshold=150M cp large-file.txt gs://my-bucket-name/

# Upload directory
gsutil cp -r local-directory gs://my-bucket-name/

# Download file
gsutil cp gs://my-bucket-name/file.txt .

# Download directory
gsutil cp -r gs://my-bucket-name/directory .

# Copy between buckets
gsutil cp gs://bucket-1/file.txt gs://bucket-2/

# Move file
gsutil mv gs://my-bucket-name/old-file.txt gs://my-bucket-name/new-file.txt

# Delete file
gsutil rm gs://my-bucket-name/file.txt

# Delete directory
gsutil rm -r gs://my-bucket-name/directory

# Sync local to bucket
gsutil rsync -r local-directory gs://my-bucket-name/directory

# Sync bucket to local
gsutil rsync -r gs://my-bucket-name/directory local-directory

# Sync with delete
gsutil rsync -d -r local-directory gs://my-bucket-name/directory

# Get file metadata
gsutil stat gs://my-bucket-name/file.txt

# Set metadata
gsutil setmeta -h "Content-Type:text/plain" gs://my-bucket-name/file.txt

# Cat file
gsutil cat gs://my-bucket-name/file.txt

# Get file hash
gsutil hash local-file.txt
```

### Access Control
```bash
# Get bucket IAM policy
gsutil iam get gs://my-bucket-name

# Set bucket IAM policy
gsutil iam set policy.json gs://my-bucket-name

# Add IAM policy binding
gsutil iam ch user:user@example.com:objectViewer gs://my-bucket-name

# Remove IAM policy binding
gsutil iam ch -d user:user@example.com:objectViewer gs://my-bucket-name

# Make bucket public
gsutil iam ch allUsers:objectViewer gs://my-bucket-name

# Get ACL
gsutil acl get gs://my-bucket-name/file.txt

# Set ACL
gsutil acl set private gs://my-bucket-name/file.txt

# Make file public
gsutil acl ch -u AllUsers:R gs://my-bucket-name/file.txt

# Set default ACL for bucket
gsutil defacl set public-read gs://my-bucket-name
```

### Bucket Configuration
```bash
# Enable versioning
gsutil versioning set on gs://my-bucket-name

# Get versioning status
gsutil versioning get gs://my-bucket-name

# Set lifecycle policy
gsutil lifecycle set lifecycle.json gs://my-bucket-name

# Get lifecycle policy
gsutil lifecycle get gs://my-bucket-name

# Enable uniform bucket-level access
gsutil uniformbucketlevelaccess set on gs://my-bucket-name

# Get uniform bucket-level access status
gsutil uniformbucketlevelaccess get gs://my-bucket-name

# Set CORS configuration
gsutil cors set cors.json gs://my-bucket-name

# Get CORS configuration
gsutil cors get gs://my-bucket-name

# Enable website configuration
gsutil web set -m index.html -e 404.html gs://my-bucket-name

# Get website configuration
gsutil web get gs://my-bucket-name

# Set labels
gsutil label set labels.json gs://my-bucket-name

# Get labels
gsutil label get gs://my-bucket-name

# Change storage class
gsutil rewrite -s nearline gs://my-bucket-name/file.txt
```

### Signed URLs
```bash
# Create signed URL (requires service account key)
gsutil signurl -d 10m private-key.json gs://my-bucket-name/file.txt

# Create signed URL with specific method
gsutil signurl -m GET -d 1h private-key.json gs://my-bucket-name/file.txt
```

---

## GKE (Google Kubernetes Engine)

### Cluster Management
```bash
# List clusters
gcloud container clusters list

# Create cluster
gcloud container clusters create cluster-1 \
  --zone=us-central1-a \
  --num-nodes=3

# Create cluster with machine type
gcloud container clusters create cluster-1 \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --num-nodes=3

# Create autopilot cluster
gcloud container clusters create-auto cluster-1 \
  --region=us-central1

# Get cluster credentials
gcloud container clusters get-credentials cluster-1 \
  --zone=us-central1-a

# Describe cluster
gcloud container clusters describe cluster-1 \
  --zone=us-central1-a

# Resize cluster
gcloud container clusters resize cluster-1 \
  --num-nodes=5 \
  --zone=us-central1-a

# Update cluster
gcloud container clusters update cluster-1 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-central1-a

# Upgrade cluster
gcloud container clusters upgrade cluster-1 \
  --master \
  --zone=us-central1-a

# Delete cluster
gcloud container clusters delete cluster-1 \
  --zone=us-central1-a
```

### Node Pool Management
```bash
# List node pools
gcloud container node-pools list --cluster=cluster-1 \
  --zone=us-central1-a

# Create node pool
gcloud container node-pools create pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --num-nodes=2

# Create preemptible node pool
gcloud container node-pools create pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a \
  --preemptible \
  --num-nodes=2

# Describe node pool
gcloud container node-pools describe pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a

# Resize node pool
gcloud container node-pools resize pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a \
  --num-nodes=4

# Update node pool
gcloud container node-pools update pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5

# Delete node pool
gcloud container node-pools delete pool-1 \
  --cluster=cluster-1 \
  --zone=us-central1-a
```

### Operations
```bash
# List operations
gcloud container operations list

# Describe operation
gcloud container operations describe operation-id \
  --zone=us-central1-a

# Wait for operation
gcloud container operations wait operation-id \
  --zone=us-central1-a
```

---

## Cloud SQL

### Instance Management
```bash
# List instances
gcloud sql instances list

# Create MySQL instance
gcloud sql instances create mysql-instance-1 \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-1 \
  --region=us-central1

# Create PostgreSQL instance
gcloud sql instances create postgres-instance-1 \
  --database-version=POSTGRES_14 \
  --tier=db-n1-standard-1 \
  --region=us-central1

# Describe instance
gcloud sql instances describe mysql-instance-1

# Patch instance (update configuration)
gcloud sql instances patch mysql-instance-1 \
  --tier=db-n1-standard-2

# Restart instance
gcloud sql instances restart mysql-instance-1

# Delete instance
gcloud sql instances delete mysql-instance-1

# Set root password
gcloud sql users set-password root \
  --host=% \
  --instance=mysql-instance-1 \
  --password=MyPassword123
```

### Database Management
```bash
# List databases
gcloud sql databases list --instance=mysql-instance-1

# Create database
gcloud sql databases create mydb \
  --instance=mysql-instance-1

# Delete database
gcloud sql databases delete mydb \
  --instance=mysql-instance-1

# Describe database
gcloud sql databases describe mydb \
  --instance=mysql-instance-1
```

### User Management
```bash
# List users
gcloud sql users list --instance=mysql-instance-1

# Create user
gcloud sql users create myuser \
  --instance=mysql-instance-1 \
  --password=MyPassword123

# Set password
gcloud sql users set-password myuser \
  --instance=mysql-instance-1 \
  --password=NewPassword456

# Delete user
gcloud sql users delete myuser \
  --instance=mysql-instance-1
```

### Backup and Restore
```bash
# List backups
gcloud sql backups list --instance=mysql-instance-1

# Create backup
gcloud sql backups create \
  --instance=mysql-instance-1

# Describe backup
gcloud sql backups describe backup-id \
  --instance=mysql-instance-1

# Restore from backup
gcloud sql backups restore backup-id \
  --backup-instance=mysql-instance-1 \
  --backup-id=backup-id

# Export database
gcloud sql export sql mysql-instance-1 \
  gs://my-bucket/export.sql \
  --database=mydb

# Import database
gcloud sql import sql mysql-instance-1 \
  gs://my-bucket/import.sql \
  --database=mydb
```

### Operations
```bash
# List operations
gcloud sql operations list --instance=mysql-instance-1

# Describe operation
gcloud sql operations describe operation-id

# Wait for operation
gcloud sql operations wait operation-id
```

---

## IAM (Identity and Access Management)

### Service Accounts
```bash
# List service accounts
gcloud iam service-accounts list

# Create service account
gcloud iam service-accounts create my-service-account \
  --display-name="My Service Account"

# Describe service account
gcloud iam service-accounts describe my-service-account@my-project.iam.gserviceaccount.com

# Delete service account
gcloud iam service-accounts delete my-service-account@my-project.iam.gserviceaccount.com

# Create key for service account
gcloud iam service-accounts keys create key.json \
  --iam-account=my-service-account@my-project.iam.gserviceaccount.com

# List keys
gcloud iam service-accounts keys list \
  --iam-account=my-service-account@my-project.iam.gserviceaccount.com

# Delete key
gcloud iam service-accounts keys delete key-id \
  --iam-account=my-service-account@my-project.iam.gserviceaccount.com
```

### IAM Policies
```bash
# Get IAM policy
gcloud projects get-iam-policy my-project-id

# Add IAM policy binding
gcloud projects add-iam-policy-binding my-project-id \
  --member=user:user@example.com \
  --role=roles/viewer

# Add IAM policy binding for service account
gcloud projects add-iam-policy-binding my-project-id \
  --member=serviceAccount:my-service-account@my-project.iam.gserviceaccount.com \
  --role=roles/editor

# Remove IAM policy binding
gcloud projects remove-iam-policy-binding my-project-id \
  --member=user:user@example.com \
  --role=roles/viewer

# Set IAM policy
gcloud projects set-iam-policy my-project-id policy.json

# Get IAM policy for resource
gcloud compute instances get-iam-policy instance-1 \
  --zone=us-central1-a

# Add IAM policy binding to resource
gcloud compute instances add-iam-policy-binding instance-1 \
  --zone=us-central1-a \
  --member=user:user@example.com \
  --role=roles/compute.instanceAdmin
```

### Roles
```bash
# List roles
gcloud iam roles list

# List predefined roles
gcloud iam roles list --filter="name:roles/"

# List custom roles
gcloud iam roles list --project=my-project-id

# Describe role
gcloud iam roles describe roles/compute.admin

# Create custom role
gcloud iam roles create myCustomRole \
  --project=my-project-id \
  --title="My Custom Role" \
  --description="Custom role for specific permissions" \
  --permissions=compute.instances.get,compute.instances.list

# Update custom role
gcloud iam roles update myCustomRole \
  --project=my-project-id \
  --add-permissions=compute.instances.start

# Delete custom role
gcloud iam roles delete myCustomRole \
  --project=my-project-id

# Undelete custom role
gcloud iam roles undelete myCustomRole \
  --project=my-project-id
```

---

## VPC (Virtual Private Cloud)

### Network Management
```bash
# List networks
gcloud compute networks list

# Create auto mode network
gcloud compute networks create my-network \
  --subnet-mode=auto

# Create custom mode network
gcloud compute networks create my-network \
  --subnet-mode=custom

# Delete network
gcloud compute networks delete my-network

# Describe network
gcloud compute networks describe my-network

# Update network
gcloud compute networks update my-network \
  --switch-to-custom-subnet-mode
```

### Subnet Management
```bash
# List subnets
gcloud compute networks subnets list

# Create subnet
gcloud compute networks subnets create my-subnet \
  --network=my-network \
  --region=us-central1 \
  --range=10.0.1.0/24

# Delete subnet
gcloud compute networks subnets delete my-subnet \
  --region=us-central1

# Describe subnet
gcloud compute networks subnets describe my-subnet \
  --region=us-central1

# Expand subnet range
gcloud compute networks subnets expand-ip-range my-subnet \
  --region=us-central1 \
  --prefix-length=20

# Update subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access
```

### Routes
```bash
# List routes
gcloud compute routes list

# Create route
gcloud compute routes create my-route \
  --network=my-network \
  --destination-range=0.0.0.0/0 \
  --next-hop-gateway=default-internet-gateway

# Delete route
gcloud compute routes delete my-route

# Describe route
gcloud compute routes describe my-route
```

### VPC Peering
```bash
# Create VPC peering
gcloud compute networks peerings create peering-1 \
  --network=network-1 \
  --peer-project=peer-project-id \
  --peer-network=network-2

# List VPC peerings
gcloud compute networks peerings list

# Delete VPC peering
gcloud compute networks peerings delete peering-1 \
  --network=network-1

# Update VPC peering
gcloud compute networks peerings update peering-1 \
  --network=network-1 \
  --import-custom-routes \
  --export-custom-routes
```

### Cloud NAT
```bash
# Create Cloud Router
gcloud compute routers create my-router \
  --network=my-network \
  --region=us-central1

# Create Cloud NAT
gcloud compute routers nats create my-nat \
  --router=my-router \
  --region=us-central1 \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges

# List NATs
gcloud compute routers nats list --router=my-router \
  --region=us-central1

# Delete NAT
gcloud compute routers nats delete my-nat \
  --router=my-router \
  --region=us-central1

# Delete router
gcloud compute routers delete my-router \
  --region=us-central1
```

### VPN
```bash
# Create VPN gateway
gcloud compute target-vpn-gateways create vpn-gateway-1 \
  --network=my-network \
  --region=us-central1

# Create VPN tunnel
gcloud compute vpn-tunnels create vpn-tunnel-1 \
  --target-vpn-gateway=vpn-gateway-1 \
  --region=us-central1 \
  --peer-address=203.0.113.1 \
  --shared-secret=MySharedSecret123

# List VPN tunnels
gcloud compute vpn-tunnels list

# Delete VPN tunnel
gcloud compute vpn-tunnels delete vpn-tunnel-1 \
  --region=us-central1

# Delete VPN gateway
gcloud compute target-vpn-gateways delete vpn-gateway-1 \
  --region=us-central1
```

---

## Cloud Functions

### Function Management
```bash
# List functions
gcloud functions list

# Deploy function (HTTP trigger)
gcloud functions deploy my-function \
  --runtime=python39 \
  --trigger-http \
  --entry-point=hello_world \
  --source=.

# Deploy function (Pub/Sub trigger)
gcloud functions deploy my-function \
  --runtime=python39 \
  --trigger-topic=my-topic \
  --entry-point=hello_pubsub \
  --source=.

# Deploy function (Cloud Storage trigger)
gcloud functions deploy my-function \
  --runtime=python39 \
  --trigger-resource=my-bucket \
  --trigger-event=google.storage.object.finalize \
  --entry-point=hello_gcs \
  --source=.

# Deploy function with environment variables
gcloud functions deploy my-function \
  --runtime=python39 \
  --trigger-http \
  --set-env-vars=KEY1=value1,KEY2=value2

# Deploy function with service account
gcloud functions deploy my-function \
  --runtime=python39 \
  --trigger-http \
  --service-account=my-service-account@my-project.iam.gserviceaccount.com

# Describe function
gcloud functions describe my-function

# Call function
gcloud functions call my-function --data='{"key":"value"}'

# Get logs
gcloud functions logs read my-function

# Get logs (follow)
gcloud functions logs read my-function --limit=10 --follow

# Delete function
gcloud functions delete my-function

# Get IAM policy
gcloud functions get-iam-policy my-function

# Add IAM policy binding
gcloud functions add-iam-policy-binding my-function \
  --member=user:user@example.com \
  --role=roles/cloudfunctions.invoker
```

---

## App Engine

### Application Management
```bash
# Deploy application
gcloud app deploy

# Deploy with specific version
gcloud app deploy --version=v1

# Deploy without promoting to live
gcloud app deploy --no-promote

# Describe application
gcloud app describe

# Browse application
gcloud app browse

# Open specific service
gcloud app browse --service=my-service

# View logs
gcloud app logs tail -s default

# Read logs
gcloud app logs read
```

### Version Management
```bash
# List versions
gcloud app versions list

# Describe version
gcloud app versions describe v1 --service=default

# Start version
gcloud app versions start v1 --service=default

# Stop version
gcloud app versions stop v1 --service=default

# Delete version
gcloud app versions delete v1 --service=default

# Migrate traffic
gcloud app services set-traffic default --splits=v1=0.9,v2=0.1

# Migrate all traffic to version
gcloud app services set-traffic default --splits=v2=1
```

### Service Management
```bash
# List services
gcloud app services list

# Describe service
gcloud app services describe default

# Delete service
gcloud app services delete my-service

# Set traffic split
gcloud app services set-traffic my-service --splits=v1=0.5,v2=0.5
```

### Instances Management
```bash
# List instances
gcloud app instances list

# Describe instance
gcloud app instances describe instance-id --service=default --version=v1

# Delete instance
gcloud app instances delete instance-id --service=default --version=v1

# SSH into instance
gcloud app instances ssh instance-id --service=default --version=v1
```

---

## BigQuery

### Dataset Management
```bash
# List datasets
bq ls

# List datasets with project
bq ls my-project:

# Create dataset
bq mk my_dataset

# Create dataset with location
bq mk --location=US my_dataset

# Delete dataset
bq rm -r my_dataset

# Show dataset info
bq show my_dataset

# Update dataset description
bq update --description "My dataset description" my_dataset

# Copy dataset
bq cp source_dataset destination_dataset
```

### Table Management
```bash
# List tables in dataset
bq ls my_dataset

# Show table schema
bq show my_dataset.my_table

# Create table
bq mk --table my_dataset.my_table schema.json

# Create table from query
bq query --destination_table=my_dataset.my_table --use_legacy_sql=false \
  'SELECT * FROM `my_dataset.source_table`'

# Load data from CSV
bq load --source_format=CSV my_dataset.my_table gs://my-bucket/data.csv schema.json

# Load data from JSON
bq load --source_format=NEWLINE_DELIMITED_JSON my_dataset.my_table gs://my-bucket/data.json

# Extract table to Cloud Storage
bq extract my_dataset.my_table gs://my-bucket/export.csv

# Delete table
bq rm my_dataset.my_table

# Copy table
bq cp my_dataset.source_table my_dataset.destination_table

# Update table schema
bq update my_dataset.my_table schema.json

# Set table expiration
bq update --expiration 3600 my_dataset.my_table
```

### Query Operations
```bash
# Run query
bq query --use_legacy_sql=false \
  'SELECT * FROM `my_dataset.my_table` LIMIT 10'

# Run query and save results
bq query --use_legacy_sql=false --destination_table=my_dataset.results \
  'SELECT * FROM `my_dataset.my_table`'

# Run query with parameters
bq query --use_legacy_sql=false --parameter=name:STRING:John \
  'SELECT * FROM `my_dataset.my_table` WHERE name = @name'

# Run dry run
bq query --dry_run --use_legacy_sql=false \
  'SELECT * FROM `my_dataset.my_table`'

# Get query results
bq show -j job_id
```

### Job Management
```bash
# List jobs
bq ls -j

# Show job info
bq show -j job_id

# Wait for job
bq wait job_id

# Cancel job
bq cancel job_id
```

---

## Cloud Build

### Build Management
```bash
# Submit build
gcloud builds submit --config=cloudbuild.yaml .

# Submit build from source
gcloud builds submit --tag=gcr.io/my-project/my-image:tag .

# List builds
gcloud builds list

# Describe build
gcloud builds describe build-id

# Get build logs
gcloud builds log build-id

# Stream build logs
gcloud builds log --stream build-id

# Cancel build
gcloud builds cancel build-id
```

### Trigger Management
```bash
# List triggers
gcloud builds triggers list

# Create trigger
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern=^main$ \
  --build-config=cloudbuild.yaml

# Describe trigger
gcloud builds triggers describe trigger-id

# Run trigger
gcloud builds triggers run trigger-id

# Update trigger
gcloud builds triggers update trigger-id \
  --branch-pattern=^master$

# Delete trigger
gcloud builds triggers delete trigger-id
```

---

## Additional Services

### Cloud Pub/Sub
```bash
# List topics
gcloud pubsub topics list

# Create topic
gcloud pubsub topics create my-topic

# Publish message
gcloud pubsub topics publish my-topic --message="Hello World"

# Delete topic
gcloud pubsub topics delete my-topic

# List subscriptions
gcloud pubsub subscriptions list

# Create subscription
gcloud pubsub subscriptions create my-subscription --topic=my-topic

# Pull messages
gcloud pubsub subscriptions pull my-subscription --auto-ack

# Delete subscription
gcloud pubsub subscriptions delete my-subscription
```

### Cloud DNS
```bash
# List managed zones
gcloud dns managed-zones list

# Create managed zone
gcloud dns managed-zones create my-zone \
  --dns-name=example.com. \
  --description="My DNS zone"

# List record sets
gcloud dns record-sets list --zone=my-zone

# Add record
gcloud dns record-sets create www.example.com. \
  --type=A \
  --ttl=300 \
  --rrdatas=203.0.113.1 \
  --zone=my-zone

# Delete record
gcloud dns record-sets delete www.example.com. \
  --type=A \
  --zone=my-zone

# Delete zone
gcloud dns managed-zones delete my-zone
```

### Cloud Monitoring
```bash
# List metric descriptors
gcloud monitoring metrics-descriptors list

# List time series
gcloud monitoring time-series list \
  --filter='metric.type="compute.googleapis.com/instance/cpu/utilization"'

# Create alert policy
gcloud alpha monitoring policies create --policy-from-file=policy.yaml

# List alert policies
gcloud alpha monitoring policies list
```

### Cloud Logging
```bash
# Read logs
gcloud logging read "resource.type=gce_instance" --limit=10

# Read logs with filter
gcloud logging read "severity>=ERROR" --limit=10

# Read logs with time range
gcloud logging read "timestamp>\"2024-01-01T00:00:00Z\"" --limit=10

# Write log entry
gcloud logging write my-log "Log message" --severity=INFO

# List logs
gcloud logging logs list

# Delete log
gcloud logging logs delete my-log
```

### Secret Manager
```bash
# Create secret
echo -n "my-secret-value" | gcloud secrets create my-secret --data-file=-

# List secrets
gcloud secrets list

# Add secret version
echo -n "new-secret-value" | gcloud secrets versions add my-secret --data-file=-

# Access secret version
gcloud secrets versions access latest --secret=my-secret

# List secret versions
gcloud secrets versions list my-secret

# Delete secret version
gcloud secrets versions destroy 1 --secret=my-secret

# Delete secret
gcloud secrets delete my-secret
```

### Cloud Scheduler
```bash
# Create job
gcloud scheduler jobs create http my-job \
  --schedule="0 2 * * *" \
  --uri="https://example.com/endpoint" \
  --http-method=POST

# List jobs
gcloud scheduler jobs list

# Run job
gcloud scheduler jobs run my-job

# Update job
gcloud scheduler jobs update http my-job \
  --schedule="0 3 * * *"

# Pause job
gcloud scheduler jobs pause my-job

# Resume job
gcloud scheduler jobs resume my-job

# Delete job
gcloud scheduler jobs delete my-job
```

---

## Tips and Best Practices

### Format Output
```bash
# JSON format
gcloud compute instances list --format=json

# YAML format
gcloud compute instances list --format=yaml

# Table format with specific columns
gcloud compute instances list --format="table(name,zone,machineType,status)"

# CSV format
gcloud compute instances list --format=csv

# Get specific field
gcloud compute instances list --format="value(name)"
```

### Filtering and Sorting
```bash
# Filter by field
gcloud compute instances list --filter="zone:us-central1-a"

# Filter with AND
gcloud compute instances list --filter="zone:us-central1-a AND status:RUNNING"

# Filter with OR
gcloud compute instances list --filter="zone:us-central1-a OR zone:us-west1-a"

# Sort results
gcloud compute instances list --sort-by=creationTimestamp

# Sort descending
gcloud compute instances list --sort-by=~creationTimestamp

# Limit results
gcloud compute instances list --limit=10
```

### Project Management
```bash
# Create project
gcloud projects create my-new-project --name="My New Project"

# Set project
gcloud config set project my-project-id

# Get current project
gcloud config get-value project

# List projects
gcloud projects list

# Delete project
gcloud projects delete my-project-id
```

---

This cheat sheet covers the most commonly used gcloud and gsutil commands for GCP certification preparation and daily operations. Practice these commands regularly to build proficiency with Google Cloud Platform.
