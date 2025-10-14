# Cloud CLI Command Comparison: AWS vs GCP vs Azure

A comprehensive side-by-side comparison of equivalent commands across AWS CLI, gcloud, and Azure CLI for cloud certification preparation.

## Table of Contents
- [Configuration & Authentication](#configuration--authentication)
- [Compute (Virtual Machines)](#compute-virtual-machines)
- [Storage](#storage)
- [Networking](#networking)
- [Identity & Access Management](#identity--access-management)
- [Databases](#databases)
- [Container Services](#container-services)
- [Serverless Functions](#serverless-functions)
- [Load Balancing](#load-balancing)
- [Monitoring & Logging](#monitoring--logging)
- [Resource Management](#resource-management)

---

## Configuration & Authentication

### Initial Setup

| Task | AWS CLI | GCP gcloud | Azure CLI |
|------|---------|------------|-----------|
| **Login/Authenticate** | `aws configure` | `gcloud init`<br>`gcloud auth login` | `az login` |
| **Set Default Region** | `aws configure set region us-east-1` | `gcloud config set compute/region us-central1` | `az config set defaults.location=eastus` |
| **Set Default Zone** | N/A (region-based) | `gcloud config set compute/zone us-central1-a` | N/A (region-based) |
| **View Current Config** | `aws configure list` | `gcloud config list` | `az account show` |
| **List Profiles/Accounts** | `aws configure list-profiles` | `gcloud auth list` | `az account list` |
| **Switch Profile/Project** | `aws configure --profile prod` | `gcloud config set project my-project` | `az account set --subscription "My Sub"` |
| **Get Account ID** | `aws sts get-caller-identity` | `gcloud config get-value project` | `az account show --query id` |

### Output Formats

| Format | AWS CLI | GCP gcloud | Azure CLI |
|--------|---------|------------|-----------|
| **JSON** | `--output json` | `--format=json` | `--output json` |
| **Table** | `--output table` | `--format=table` | `--output table` |
| **Text** | `--output text` | `--format=value(field)` | `--output tsv` |
| **YAML** | `--output yaml` | `--format=yaml` | `--output yaml` |

---

## Compute (Virtual Machines)

### Instance/VM Management

| Task | AWS CLI | GCP gcloud | Azure CLI |
|------|---------|------------|-----------|
| **List All Instances** | `aws ec2 describe-instances` | `gcloud compute instances list` | `az vm list` |
| **List Running Instances** | `aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"` | `gcloud compute instances list --filter="status:RUNNING"` | `az vm list --query "[?powerState=='VM running']"` |
| **Create Instance** | `aws ec2 run-instances --image-id ami-xxx --instance-type t3.micro` | `gcloud compute instances create vm1 --machine-type=e2-micro --image-family=debian-11` | `az vm create --name vm1 --image UbuntuLTS --size Standard_B1s` |
| **Start Instance** | `aws ec2 start-instances --instance-ids i-xxx` | `gcloud compute instances start vm1` | `az vm start --name vm1` |
| **Stop Instance** | `aws ec2 stop-instances --instance-ids i-xxx` | `gcloud compute instances stop vm1` | `az vm stop --name vm1` |
| **Restart Instance** | `aws ec2 reboot-instances --instance-ids i-xxx` | `gcloud compute instances reset vm1` | `az vm restart --name vm1` |
| **Delete Instance** | `aws ec2 terminate-instances --instance-ids i-xxx` | `gcloud compute instances delete vm1` | `az vm delete --name vm1` |
| **Get Instance Details** | `aws ec2 describe-instances --instance-ids i-xxx` | `gcloud compute instances describe vm1` | `az vm show --name vm1` |

### SSH and Remote Access

| Task | AWS CLI | GCP gcloud | Azure CLI |
|------|---------|------------|-----------|
| **SSH to Instance** | `ssh -i key.pem user@ip-address` | `gcloud compute ssh vm1` | `az vm ssh --name vm1` (if Azure AD configured)<br>or `ssh user@ip-address` |
| **Get Instance IP** | `aws ec2 describe-instances --instance-ids i-xxx --query 'Reservations[0].Instances[0].PublicIpAddress'` | `gcloud compute instances describe vm1 --format='get(networkInterfaces[0].accessConfigs[0].natIP)'` | `az vm list-ip-addresses --name vm1` |
| **Execute Remote Command** | Use SSM or SSH | `gcloud compute ssh vm1 --command="ls -la"` | `az vm run-command invoke --name vm1 --command-id RunShellScript --scripts "ls -la"` |

### Instance Sizing

| Task | AWS CLI | GCP gcloud | Azure CLI |
|------|---------|------------|-----------|
| **List Available Sizes** | `aws ec2 describe-instance-types` | `gcloud compute machine-types list` | `az vm list-sizes --location eastus` |
| **Resize Instance** | `aws ec2 modify-instance-attribute --instance-id i-xxx --instance-type "{\"Value\":\"t3.medium\"}"` (requires stop) | `gcloud compute instances set-machine-type vm1 --machine-type=e2-medium` (requires stop) | `az vm resize --name vm1 --size Standard_DS2_v2` (requires stop) |

### Images and Snapshots

| Task | AWS CLI | GCP gcloud | Azure CLI |
|------|---------|------------|-----------|
| **List Images** | `aws ec2 describe-images --owners self` | `gcloud compute images list` | `az image list` |
| **Create Image** | `aws ec2 create-image --instance-id i-xxx --name "MyImage"` | `gcloud compute images create my-image --source-disk=disk1` | `az image create --name myImage --source vm1` |
| **Delete Image** | `aws ec2 deregister-image --image-id ami-xxx` | `gcloud compute images delete my-image` | `az image delete --name myImage` |
| **Create Snapshot** | `aws ec2 create-snapshot --volume-id vol-xxx` | `gcloud compute disks snapshot disk1 --snapshot-names=snap1` | `az snapshot create --name snap1 --source disk1` |

---

## Storage

### Object Storage (Buckets/Containers)

| Task | AWS S3 | GCP Cloud Storage | Azure Blob Storage |
|------|--------|-------------------|-------------------|
| **List Buckets** | `aws s3 ls` | `gsutil ls` | `az storage account list` |
| **Create Bucket** | `aws s3 mb s3://my-bucket` | `gsutil mb gs://my-bucket` | `az storage account create --name mystorageacct`<br>`az storage container create --name mycontainer` |
| **Delete Bucket** | `aws s3 rb s3://my-bucket` | `gsutil rb gs://my-bucket` | `az storage container delete --name mycontainer` |
| **Upload File** | `aws s3 cp file.txt s3://my-bucket/` | `gsutil cp file.txt gs://my-bucket/` | `az storage blob upload --container mycontainer --file file.txt --name file.txt` |
| **Download File** | `aws s3 cp s3://my-bucket/file.txt .` | `gsutil cp gs://my-bucket/file.txt .` | `az storage blob download --container mycontainer --name file.txt --file file.txt` |
| **Upload Directory** | `aws s3 cp mydir s3://my-bucket/mydir --recursive` | `gsutil cp -r mydir gs://my-bucket/` | `az storage blob upload-batch --destination mycontainer --source mydir` |
| **Sync Directory** | `aws s3 sync mydir s3://my-bucket/mydir` | `gsutil rsync -r mydir gs://my-bucket/mydir` | `az storage blob sync --container mycontainer --source mydir` |
| **List Objects** | `aws s3 ls s3://my-bucket/` | `gsutil ls gs://my-bucket/` | `az storage blob list --container mycontainer` |
| **Delete Object** | `aws s3 rm s3://my-bucket/file.txt` | `gsutil rm gs://my-bucket/file.txt` | `az storage blob delete --container mycontainer --name file.txt` |
| **Make Public** | `aws s3api put-bucket-acl --bucket my-bucket --acl public-read` | `gsutil iam ch allUsers:objectViewer gs://my-bucket` | `az storage container set-permission --name mycontainer --public-access blob` |
| **Generate Pre-signed URL** | `aws s3 presign s3://my-bucket/file.txt --expires-in 3600` | `gsutil signurl -d 1h key.json gs://my-bucket/file.txt` | `az storage blob generate-sas --container mycontainer --name file.txt --expiry 2024-12-31 --permissions r` |

### Block Storage (Disks/Volumes)

| Task | AWS EBS | GCP Persistent Disk | Azure Managed Disks |
|------|---------|---------------------|---------------------|
| **List Volumes** | `aws ec2 describe-volumes` | `gcloud compute disks list` | `az disk list` |
| **Create Volume** | `aws ec2 create-volume --size 100 --availability-zone us-east-1a --volume-type gp3` | `gcloud compute disks create disk1 --size=100GB --type=pd-standard` | `az disk create --name disk1 --size-gb 100 --sku Standard_LRS` |
| **Attach Volume** | `aws ec2 attach-volume --volume-id vol-xxx --instance-id i-xxx --device /dev/sdf` | `gcloud compute instances attach-disk vm1 --disk=disk1` | `az vm disk attach --vm-name vm1 --name disk1` |
| **Detach Volume** | `aws ec2 detach-volume --volume-id vol-xxx` | `gcloud compute instances detach-disk vm1 --disk=disk1` | `az vm disk detach --vm-name vm1 --name disk1` |
| **Delete Volume** | `aws ec2 delete-volume --volume-id vol-xxx` | `gcloud compute disks delete disk1` | `az disk delete --name disk1` |
| **Resize Volume** | `aws ec2 modify-volume --volume-id vol-xxx --size 200` | `gcloud compute disks resize disk1 --size=200GB` | `az disk update --name disk1 --size-gb 200` |

---

## Networking

### Virtual Networks

| Task | AWS VPC | GCP VPC | Azure VNet |
|------|---------|---------|------------|
| **List Networks** | `aws ec2 describe-vpcs` | `gcloud compute networks list` | `az network vnet list` |
| **Create Network** | `aws ec2 create-vpc --cidr-block 10.0.0.0/16` | `gcloud compute networks create vpc1 --subnet-mode=custom` | `az network vnet create --name vnet1 --address-prefix 10.0.0.0/16` |
| **Delete Network** | `aws ec2 delete-vpc --vpc-id vpc-xxx` | `gcloud compute networks delete vpc1` | `az network vnet delete --name vnet1` |
| **Create Subnet** | `aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24` | `gcloud compute networks subnets create subnet1 --network=vpc1 --region=us-central1 --range=10.0.1.0/24` | `az network vnet subnet create --vnet-name vnet1 --name subnet1 --address-prefix 10.0.1.0/24` |
| **List Subnets** | `aws ec2 describe-subnets` | `gcloud compute networks subnets list` | `az network vnet subnet list --vnet-name vnet1` |

### Firewall/Security Groups

| Task | AWS Security Groups | GCP Firewall Rules | Azure NSG |
|------|---------------------|-------------------|-----------|
| **List Rules** | `aws ec2 describe-security-groups` | `gcloud compute firewall-rules list` | `az network nsg list` |
| **Create Firewall/NSG** | Security groups created with rules | N/A (rules added directly to VPC) | `az network nsg create --name nsg1` |
| **Allow SSH (Port 22)** | `aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 0.0.0.0/0` | `gcloud compute firewall-rules create allow-ssh --allow=tcp:22 --source-ranges=0.0.0.0/0` | `az network nsg rule create --nsg-name nsg1 --name allow-ssh --priority 1000 --destination-port-ranges 22 --access Allow --protocol Tcp` |
| **Allow HTTP (Port 80)** | `aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 80 --cidr 0.0.0.0/0` | `gcloud compute firewall-rules create allow-http --allow=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server` | `az network nsg rule create --nsg-name nsg1 --name allow-http --priority 1001 --destination-port-ranges 80 --access Allow --protocol Tcp` |
| **Delete Rule** | `aws ec2 revoke-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 0.0.0.0/0` | `gcloud compute firewall-rules delete allow-ssh` | `az network nsg rule delete --nsg-name nsg1 --name allow-ssh` |

### Public IP Addresses

| Task | AWS Elastic IP | GCP External IP | Azure Public IP |
|------|----------------|-----------------|-----------------|
| **List Public IPs** | `aws ec2 describe-addresses` | `gcloud compute addresses list` | `az network public-ip list` |
| **Allocate IP** | `aws ec2 allocate-address --domain vpc` | `gcloud compute addresses create ip1 --region=us-central1` | `az network public-ip create --name ip1` |
| **Associate IP** | `aws ec2 associate-address --instance-id i-xxx --allocation-id eipalloc-xxx` | `gcloud compute instances add-access-config vm1 --access-config-name="External NAT" --address=<IP>` | `az network nic ip-config update --nic-name nic1 --name ipconfig1 --public-ip-address ip1` |
| **Release IP** | `aws ec2 release-address --allocation-id eipalloc-xxx` | `gcloud compute addresses delete ip1` | `az network public-ip delete --name ip1` |

### VPN

| Task | AWS VPN | GCP Cloud VPN | Azure VPN Gateway |
|------|---------|---------------|-------------------|
| **Create VPN Gateway** | `aws ec2 create-vpn-gateway --type ipsec.1` | `gcloud compute target-vpn-gateways create vpn-gateway1 --network=vpc1` | `az network vnet-gateway create --name vpn-gateway1 --gateway-type Vpn` |
| **List VPN Gateways** | `aws ec2 describe-vpn-gateways` | `gcloud compute target-vpn-gateways list` | `az network vnet-gateway list` |

---

## Identity & Access Management

### Users

| Task | AWS IAM | GCP IAM | Azure AD |
|------|---------|---------|----------|
| **List Users** | `aws iam list-users` | `gcloud projects get-iam-policy PROJECT_ID` (shows bindings) | `az ad user list` |
| **Create User** | `aws iam create-user --user-name john` | N/A (use Google Workspace) | `az ad user create --display-name "John Doe" --user-principal-name john@example.com --password Pass123!` |
| **Delete User** | `aws iam delete-user --user-name john` | N/A | `az ad user delete --id john@example.com` |
| **List Access Keys** | `aws iam list-access-keys --user-name john` | `gcloud iam service-accounts keys list --iam-account=sa@project.iam.gserviceaccount.com` | N/A (uses Azure AD tokens) |
| **Create Access Key** | `aws iam create-access-key --user-name john` | `gcloud iam service-accounts keys create key.json --iam-account=sa@project.iam.gserviceaccount.com` | `az ad sp create-for-rbac --name myApp` |

### Groups

| Task | AWS IAM | GCP IAM | Azure AD |
|------|---------|---------|----------|
| **List Groups** | `aws iam list-groups` | N/A (use Google Groups) | `az ad group list` |
| **Create Group** | `aws iam create-group --group-name developers` | N/A | `az ad group create --display-name developers --mail-nickname developers` |
| **Add User to Group** | `aws iam add-user-to-group --user-name john --group-name developers` | N/A | `az ad group member add --group developers --member-id <user-id>` |

### Roles and Policies

| Task | AWS IAM | GCP IAM | Azure RBAC |
|------|---------|---------|-----------|
| **List Roles** | `aws iam list-roles` | `gcloud iam roles list` | `az role definition list` |
| **Attach Policy to User** | `aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess` | `gcloud projects add-iam-policy-binding PROJECT_ID --member=user:john@example.com --role=roles/storage.objectViewer` | `az role assignment create --assignee john@example.com --role "Storage Blob Data Reader"` |
| **Detach Policy** | `aws iam detach-user-policy --user-name john --policy-arn <policy-arn>` | `gcloud projects remove-iam-policy-binding PROJECT_ID --member=user:john@example.com --role=roles/storage.objectViewer` | `az role assignment delete --assignee john@example.com --role "Storage Blob Data Reader"` |
| **Create Custom Role** | `aws iam create-role --role-name MyRole --assume-role-policy-document file://trust.json` | `gcloud iam roles create myRole --project=PROJECT_ID --permissions=compute.instances.get,compute.instances.list` | `az role definition create --role-definition role.json` |

### Service Accounts

| Task | AWS IAM Role | GCP Service Account | Azure Service Principal |
|------|--------------|---------------------|-------------------------|
| **List Service Accounts** | `aws iam list-roles` | `gcloud iam service-accounts list` | `az ad sp list` |
| **Create Service Account** | `aws iam create-role --role-name MyRole --assume-role-policy-document file://trust.json` | `gcloud iam service-accounts create sa-name --display-name="My Service Account"` | `az ad sp create-for-rbac --name myServicePrincipal` |
| **Grant Permissions** | `aws iam attach-role-policy --role-name MyRole --policy-arn <policy-arn>` | `gcloud projects add-iam-policy-binding PROJECT_ID --member=serviceAccount:sa@project.iam.gserviceaccount.com --role=roles/editor` | `az role assignment create --assignee <sp-id> --role Contributor` |
| **Create Key** | N/A (uses AssumeRole) | `gcloud iam service-accounts keys create key.json --iam-account=sa@project.iam.gserviceaccount.com` | `az ad sp credential reset --id <sp-id>` |
| **Delete Service Account** | `aws iam delete-role --role-name MyRole` | `gcloud iam service-accounts delete sa@project.iam.gserviceaccount.com` | `az ad sp delete --id <sp-id>` |

---

## Databases

### Relational Databases

| Task | AWS RDS | GCP Cloud SQL | Azure SQL Database |
|------|---------|---------------|-------------------|
| **List DB Instances** | `aws rds describe-db-instances` | `gcloud sql instances list` | `az sql server list` |
| **Create DB Instance** | `aws rds create-db-instance --db-instance-identifier mydb --db-instance-class db.t3.micro --engine mysql --master-username admin --master-user-password Pass123!` | `gcloud sql instances create mydb --database-version=MYSQL_8_0 --tier=db-n1-standard-1 --region=us-central1` | `az sql server create --name myserver --admin-user admin --admin-password Pass123!`<br>`az sql db create --server myserver --name mydb` |
| **Delete DB Instance** | `aws rds delete-db-instance --db-instance-identifier mydb --skip-final-snapshot` | `gcloud sql instances delete mydb` | `az sql db delete --server myserver --name mydb`<br>`az sql server delete --name myserver` |
| **Start DB Instance** | `aws rds start-db-instance --db-instance-identifier mydb` | `gcloud sql instances patch mydb --activation-policy=ALWAYS` | N/A (always running) |
| **Stop DB Instance** | `aws rds stop-db-instance --db-instance-identifier mydb` | `gcloud sql instances patch mydb --activation-policy=NEVER` | N/A (pause/resume available for some tiers) |
| **Create Snapshot** | `aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier snap1` | `gcloud sql backups create --instance=mydb` | `az sql db copy --server myserver --name mydb --dest-name mydb-copy` (copy as backup) |
| **List Snapshots** | `aws rds describe-db-snapshots` | `gcloud sql backups list --instance=mydb` | `az sql db list --server myserver` |

### NoSQL Databases

| Task | AWS DynamoDB | GCP Firestore/Datastore | Azure Cosmos DB |
|------|--------------|-------------------------|-----------------|
| **List Tables** | `aws dynamodb list-tables` | N/A (use Firestore console/API) | `az cosmosdb list` |
| **Create Table** | `aws dynamodb create-table --table-name mytable --attribute-definitions AttributeName=Id,AttributeType=S --key-schema AttributeName=Id,KeyType=HASH` | N/A (schema-less) | `az cosmosdb create --name mycosmosdb` |
| **Put Item** | `aws dynamodb put-item --table-name mytable --item '{"Id":{"S":"123"}}'` | N/A (use client libraries) | N/A (use client libraries) |
| **Get Item** | `aws dynamodb get-item --table-name mytable --key '{"Id":{"S":"123"}}'` | N/A | N/A |
| **Delete Table** | `aws dynamodb delete-table --table-name mytable` | N/A | `az cosmosdb delete --name mycosmosdb` |

---

## Container Services

### Container Registries

| Task | AWS ECR | GCP Container Registry | Azure Container Registry |
|------|---------|------------------------|-------------------------|
| **List Registries** | `aws ecr describe-repositories` | `gcloud container images list` | `az acr list` |
| **Create Registry** | `aws ecr create-repository --repository-name myrepo` | Automatic (push to `gcr.io/PROJECT_ID/IMAGE`) | `az acr create --name myregistry --sku Basic` |
| **Login to Registry** | `aws ecr get-login-password \| docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com` | `gcloud auth configure-docker` | `az acr login --name myregistry` |
| **Push Image** | `docker push <account>.dkr.ecr.<region>.amazonaws.com/myrepo:tag` | `docker push gcr.io/PROJECT_ID/myimage:tag` | `docker push myregistry.azurecr.io/myimage:tag` |
| **List Images** | `aws ecr list-images --repository-name myrepo` | `gcloud container images list --repository=gcr.io/PROJECT_ID` | `az acr repository list --name myregistry` |
| **Delete Registry** | `aws ecr delete-repository --repository-name myrepo --force` | `gcloud container images delete gcr.io/PROJECT_ID/myimage:tag` | `az acr delete --name myregistry` |

### Kubernetes Services

| Task | AWS EKS | GCP GKE | Azure AKS |
|------|---------|---------|-----------|
| **List Clusters** | `aws eks list-clusters` | `gcloud container clusters list` | `az aks list` |
| **Create Cluster** | `aws eks create-cluster --name mycluster --role-arn <role-arn> --resources-vpc-config subnetIds=<subnet-ids>` | `gcloud container clusters create mycluster --num-nodes=3` | `az aks create --name mycluster --node-count 3 --generate-ssh-keys` |
| **Get Credentials** | `aws eks update-kubeconfig --name mycluster` | `gcloud container clusters get-credentials mycluster` | `az aks get-credentials --name mycluster` |
| **Scale Cluster** | `aws eks update-nodegroup-config --cluster-name mycluster --nodegroup-name <nodegroup> --scaling-config desiredSize=5` | `gcloud container clusters resize mycluster --num-nodes=5` | `az aks scale --name mycluster --node-count 5` |
| **Delete Cluster** | `aws eks delete-cluster --name mycluster` | `gcloud container clusters delete mycluster` | `az aks delete --name mycluster` |

---

## Serverless Functions

### Function Management

| Task | AWS Lambda | GCP Cloud Functions | Azure Functions |
|------|------------|---------------------|-----------------|
| **List Functions** | `aws lambda list-functions` | `gcloud functions list` | `az functionapp list` |
| **Create Function** | `aws lambda create-function --function-name myfunc --runtime python3.9 --role <role-arn> --handler lambda_function.lambda_handler --zip-file fileb://function.zip` | `gcloud functions deploy myfunc --runtime=python39 --trigger-http --entry-point=hello_world --source=.` | `az functionapp create --name myfuncapp --storage-account mystorage --consumption-plan-location eastus --runtime python --runtime-version 3.9` |
| **Update Function Code** | `aws lambda update-function-code --function-name myfunc --zip-file fileb://function.zip` | `gcloud functions deploy myfunc --source=.` | `az functionapp deployment source config-zip --name myfuncapp --src function.zip` |
| **Invoke Function** | `aws lambda invoke --function-name myfunc --payload '{"key":"value"}' response.json` | `gcloud functions call myfunc --data='{"key":"value"}'` | `az functionapp function invoke --name myfuncapp --function-name myfunc` |
| **Get Logs** | `aws logs tail /aws/lambda/myfunc --follow` | `gcloud functions logs read myfunc` | `az functionapp log tail --name myfuncapp` |
| **Delete Function** | `aws lambda delete-function --function-name myfunc` | `gcloud functions delete myfunc` | `az functionapp delete --name myfuncapp` |

### Environment Variables

| Task | AWS Lambda | GCP Cloud Functions | Azure Functions |
|------|------------|---------------------|-----------------|
| **Set Environment Variable** | `aws lambda update-function-configuration --function-name myfunc --environment "Variables={KEY1=value1}"` | `gcloud functions deploy myfunc --set-env-vars=KEY1=value1` | `az functionapp config appsettings set --name myfuncapp --settings KEY1=value1` |
| **List Environment Variables** | `aws lambda get-function-configuration --function-name myfunc --query Environment` | `gcloud functions describe myfunc --format="value(environmentVariables)"` | `az functionapp config appsettings list --name myfuncapp` |

---

## Load Balancing

### Load Balancer Creation

| Task | AWS ELB/ALB | GCP Cloud Load Balancing | Azure Load Balancer |
|------|-------------|--------------------------|---------------------|
| **List Load Balancers** | `aws elbv2 describe-load-balancers` | `gcloud compute forwarding-rules list` | `az network lb list` |
| **Create Load Balancer** | `aws elbv2 create-load-balancer --name mylb --subnets <subnet-ids> --security-groups <sg-ids>` | `gcloud compute forwarding-rules create lb-rule --global --target-http-proxy=http-lb-proxy --ports=80` | `az network lb create --name mylb --sku Standard --vnet-name vnet1 --subnet subnet1` |
| **Create Target Group** | `aws elbv2 create-target-group --name mytargets --protocol HTTP --port 80 --vpc-id vpc-xxx` | `gcloud compute backend-services create backend-svc --protocol=HTTP --health-checks=health-check` | `az network lb address-pool create --lb-name mylb --name mypool` |
| **Register Targets** | `aws elbv2 register-targets --target-group-arn <arn> --targets Id=i-xxx Id=i-yyy` | `gcloud compute backend-services add-backend backend-svc --instance-group=mygroup --instance-group-zone=us-central1-a` | `az network nic ip-config address-pool add --nic-name nic1 --ip-config-name ipconfig1 --lb-name mylb --address-pool mypool` |
| **Delete Load Balancer** | `aws elbv2 delete-load-balancer --load-balancer-arn <arn>` | `gcloud compute forwarding-rules delete lb-rule` | `az network lb delete --name mylb` |

---

## Monitoring & Logging

### Logs

| Task | AWS CloudWatch Logs | GCP Cloud Logging | Azure Monitor Logs |
|------|---------------------|-------------------|-------------------|
| **List Log Groups** | `aws logs describe-log-groups` | `gcloud logging logs list` | `az monitor log-analytics workspace list` |
| **Create Log Group** | `aws logs create-log-group --log-group-name /app/myapp` | Automatic | `az monitor log-analytics workspace create --name myworkspace` |
| **View Logs** | `aws logs tail /app/myapp --follow` | `gcloud logging read "resource.type=gce_instance" --limit=10` | `az monitor log-analytics query --workspace myworkspace --analytics-query "AzureActivity \| limit 10"` |
| **Filter Logs** | `aws logs filter-log-events --log-group-name /app/myapp --filter-pattern "ERROR"` | `gcloud logging read "severity>=ERROR" --limit=10` | `az monitor log-analytics query --workspace myworkspace --analytics-query "AzureActivity \| where Level=='Error'"` |
| **Delete Log Group** | `aws logs delete-log-group --log-group-name /app/myapp` | `gcloud logging logs delete LOG_NAME` | `az monitor log-analytics workspace delete --name myworkspace` |

### Metrics

| Task | AWS CloudWatch | GCP Cloud Monitoring | Azure Monitor |
|------|----------------|---------------------|---------------|
| **List Metrics** | `aws cloudwatch list-metrics` | `gcloud monitoring metrics-descriptors list` | `az monitor metrics list-definitions --resource <resource-id>` |
| **Get Metric Data** | `aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-xxx --start-time 2024-01-01T00:00:00Z --end-time 2024-01-02T00:00:00Z --period 3600 --statistics Average` | `gcloud monitoring time-series list --filter='metric.type="compute.googleapis.com/instance/cpu/utilization"'` | `az monitor metrics list --resource <resource-id> --metric "Percentage CPU"` |
| **Put Custom Metric** | `aws cloudwatch put-metric-data --namespace MyApp --metric-name PageViews --value 100` | `gcloud monitoring time-series create --project=PROJECT_ID` (requires JSON payload) | `az monitor metrics` (use Azure Monitor REST API) |

### Alerts

| Task | AWS CloudWatch Alarms | GCP Cloud Monitoring Alerts | Azure Monitor Alerts |
|------|----------------------|----------------------------|---------------------|
| **List Alarms** | `aws cloudwatch describe-alarms` | `gcloud alpha monitoring policies list` | `az monitor metrics alert list` |
| **Create Alarm** | `aws cloudwatch put-metric-alarm --alarm-name high-cpu --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average --period 300 --threshold 80 --comparison-operator GreaterThanThreshold` | `gcloud alpha monitoring policies create --policy-from-file=policy.yaml` | `az monitor metrics alert create --name high-cpu --resource <resource-id> --condition "avg Percentage CPU > 80"` |
| **Delete Alarm** | `aws cloudwatch delete-alarms --alarm-names high-cpu` | `gcloud alpha monitoring policies delete POLICY_NAME` | `az monitor metrics alert delete --name high-cpu` |

---

## Resource Management

### Resource Groups/Projects

| Task | AWS | GCP | Azure |
|------|-----|-----|-------|
| **List Projects/Subscriptions** | `aws organizations list-accounts` | `gcloud projects list` | `az account list` |
| **Create Resource Group** | N/A (resources organized by tags) | `gcloud projects create PROJECT_ID --name="My Project"` | `az group create --name myResourceGroup --location eastus` |
| **Delete Resource Group** | N/A | `gcloud projects delete PROJECT_ID` | `az group delete --name myResourceGroup` |
| **List Resources** | `aws resourcegroupstaggingapi get-resources` | `gcloud asset search-all-resources` | `az resource list --resource-group myResourceGroup` |

### Tagging/Labeling

| Task | AWS Tags | GCP Labels | Azure Tags |
|------|----------|-----------|-----------|
| **Add Tags to Resource** | `aws ec2 create-tags --resources i-xxx --tags Key=Environment,Value=Production` | `gcloud compute instances add-labels vm1 --labels=environment=production` | `az resource tag --tags Environment=Production --resource-group myResourceGroup --name vm1 --resource-type Microsoft.Compute/virtualMachines` |
| **List Resources by Tag** | `aws resourcegroupstaggingapi get-resources --tag-filters Key=Environment,Values=Production` | `gcloud compute instances list --filter="labels.environment=production"` | `az resource list --tag Environment=Production` |
| **Remove Tags** | `aws ec2 delete-tags --resources i-xxx --tags Key=Environment` | `gcloud compute instances remove-labels vm1 --labels=environment` | `az resource tag --tags "" --resource-group myResourceGroup --name vm1 --resource-type Microsoft.Compute/virtualMachines` |

### Cost Management

| Task | AWS | GCP | Azure |
|------|-----|-----|-------|
| **Get Cost Report** | `aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-01-31 --granularity MONTHLY --metrics BlendedCost` | `gcloud billing accounts list` (billing export to BigQuery recommended) | `az consumption usage list --start-date 2024-01-01 --end-date 2024-01-31` |
| **Create Budget** | `aws budgets create-budget --account-id <account-id> --budget file://budget.json` | `gcloud billing budgets create --billing-account=<account-id> --display-name="My Budget" --budget-amount=1000` | `az consumption budget create --budget-name myBudget --amount 1000 --category cost` |

### Regions and Availability

| Task | AWS | GCP | Azure |
|------|-----|-----|-------|
| **List Regions** | `aws ec2 describe-regions` | `gcloud compute regions list` | `az account list-locations` |
| **List Availability Zones** | `aws ec2 describe-availability-zones` | `gcloud compute zones list` | N/A (uses regions) |

---

## Quick Reference: Service Name Mapping

| Service Type | AWS | GCP | Azure |
|-------------|-----|-----|-------|
| **Compute** | EC2 | Compute Engine | Virtual Machines |
| **Object Storage** | S3 | Cloud Storage | Blob Storage |
| **Block Storage** | EBS | Persistent Disk | Managed Disks |
| **File Storage** | EFS | Filestore | Azure Files |
| **Networking** | VPC | VPC | Virtual Network (VNet) |
| **Load Balancer** | ELB/ALB/NLB | Cloud Load Balancing | Load Balancer / Application Gateway |
| **CDN** | CloudFront | Cloud CDN | Azure CDN |
| **DNS** | Route 53 | Cloud DNS | Azure DNS |
| **IAM** | IAM | Cloud IAM | Azure AD / RBAC |
| **Container Registry** | ECR | Container Registry (GCR) | Container Registry (ACR) |
| **Kubernetes** | EKS | GKE | AKS |
| **Serverless Functions** | Lambda | Cloud Functions | Azure Functions |
| **PaaS** | Elastic Beanstalk | App Engine | App Service |
| **Relational DB** | RDS | Cloud SQL | SQL Database |
| **NoSQL DB** | DynamoDB | Firestore/Datastore | Cosmos DB |
| **Data Warehouse** | Redshift | BigQuery | Synapse Analytics |
| **Message Queue** | SQS | Cloud Pub/Sub | Service Bus Queue |
| **Pub/Sub Messaging** | SNS | Cloud Pub/Sub | Service Bus Topic |
| **Monitoring** | CloudWatch | Cloud Monitoring | Azure Monitor |
| **Logging** | CloudWatch Logs | Cloud Logging | Azure Monitor Logs |
| **Secret Management** | Secrets Manager | Secret Manager | Key Vault |
| **Infrastructure as Code** | CloudFormation | Deployment Manager | ARM Templates / Bicep |
| **CI/CD** | CodePipeline | Cloud Build | Azure DevOps / Pipelines |

---

## Command Syntax Patterns

### Common Patterns Across Clouds

**AWS CLI Pattern:**
```bash
aws <service> <operation> --parameter value
```

**GCP gcloud Pattern:**
```bash
gcloud <service> <resource> <operation> --flag=value
```

**Azure CLI Pattern:**
```bash
az <service> <resource> <operation> --parameter value
```

### Filter and Query Patterns

**AWS (JMESPath):**
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' --output table
```

**GCP (gcloud filters):**
```bash
gcloud compute instances list --filter="status:RUNNING" --format="table(name,zone,status)"
```

**Azure (JMESPath):**
```bash
az vm list --query "[?powerState=='VM running'].{Name:name, Location:location}" --output table
```

---

## Tips for Multi-Cloud Certification

### Study Strategies

1. **Learn One Cloud First**: Master one cloud provider deeply, then map concepts to other clouds
2. **Focus on Concepts**: Understand the underlying concepts (compute, storage, networking) rather than memorizing commands
3. **Use Comparison Tables**: Study service mappings and equivalent commands side-by-side
4. **Hands-On Practice**: Create equivalent architectures across all three clouds
5. **Understand Pricing Models**: Each cloud has different pricing structures for the same services
6. **Know the Differences**: Some services have unique features (e.g., AWS IAM Roles vs GCP Service Accounts vs Azure Service Principals)

### Key Differences to Remember

| Concept | AWS | GCP | Azure |
|---------|-----|-----|-------|
| **Organizational Unit** | Account | Project | Subscription / Resource Group |
| **Default Networking** | VPC required | Auto-mode VPC by default | VNet required |
| **Regions/Zones** | Regions + Availability Zones | Regions + Zones | Regions (+ Availability Zones in select regions) |
| **CLI Tool** | `aws` | `gcloud` + `gsutil` | `az` |
| **Configuration** | `~/.aws/credentials` | `~/.config/gcloud` | `~/.azure` |
| **Authentication** | Access Keys | OAuth 2.0 / Service Accounts | Azure AD / Service Principals |
| **Pricing Model** | Pay-as-you-go | Pay-as-you-go | Pay-as-you-go |
| **Free Tier Duration** | 12 months | 90 days (trial credit) | 12 months |

### Common Certification Topics

All three clouds test on these core concepts:
- Compute (VMs, containers, serverless)
- Storage (object, block, file)
- Networking (VPC, subnets, firewalls, load balancers)
- Identity and Access Management
- Databases (relational and NoSQL)
- Monitoring and logging
- Cost management
- High availability and disaster recovery
- Security best practices
- Infrastructure as Code

---

This comparison cheat sheet helps you quickly translate your knowledge from one cloud provider to another, making multi-cloud certification preparation more efficient and effective.
