# Infrastructure Automation - GCP Professional Cloud DevOps Engineer

## Overview

Infrastructure as Code, configuration management, and automation practices for managing GCP infrastructure at scale.

## Infrastructure as Code

### Terraform
**Module Structure**:
```hcl
# variables.tf
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "Default region"
  type        = string
  default     = "us-central1"
}

# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "terraform-state-bucket"
    prefix = "prod"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Resources
resource "google_compute_network" "vpc" {
  name                    = "production-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "production-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id
}

# outputs.tf
output "network_name" {
  value = google_compute_network.vpc.name
}
```

### Cloud Deployment Manager
```yaml
imports:
- path: network.jinja

resources:
- name: production-network
  type: network.jinja
  properties:
    name: prod-vpc
    autoCreateSubnetworks: false

- name: gke-cluster
  type: container.v1.cluster
  properties:
    zone: us-central1-a
    cluster:
      name: production-cluster
      initialNodeCount: 3
      nodeConfig:
        machineType: n1-standard-4
```

## Configuration Management

### Ansible for GCP
**Playbook Example**:
```yaml
---
- name: Configure GCP infrastructure
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Create GCE instance
      gcp_compute_instance:
        name: web-server
        machine_type: n1-standard-2
        zone: us-central1-a
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: projects/debian-cloud/global/images/family/debian-11
        network_interfaces:
          - network: "{{ vpc_network }}"
            access_configs:
              - name: External NAT
                type: ONE_TO_ONE_NAT
        project: "{{ gcp_project }}"
        auth_kind: serviceaccount
        service_account_file: "{{ gcp_cred_file }}"
```

### Cloud Init for VM Startup
```yaml
#cloud-config
packages:
  - nginx
  - docker.io

runcmd:
  - systemctl start docker
  - docker run -d -p 80:8080 myapp:latest
  - systemctl enable nginx

write_files:
  - path: /etc/nginx/nginx.conf
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:8080;
        }
      }
```

## Best Practices

### Infrastructure as Code
1. **Version Control**: Store all IaC in Git
2. **Modular Design**: Reusable, composable modules
3. **Environment Separation**: Dev, staging, prod
4. **State Management**: Remote backend with locking
5. **Testing**: Validate before apply
6. **Documentation**: Clear README and variables
7. **Review Process**: Pull request reviews
8. **Drift Detection**: Regular state checks

### Automation
1. **Idempotency**: Safe to run multiple times
2. **Error Handling**: Graceful failure and retry
3. **Logging**: Detailed execution logs
4. **Notifications**: Alert on failures
5. **Rollback**: Automated rollback on errors
6. **Testing**: Test automation scripts
7. **Security**: Secure credential handling

## Study Tips

1. Practice Terraform and Deployment Manager
2. Understand state management
3. Know module patterns
4. Configuration management tools
5. Automation best practices

## Additional Resources

- [Terraform on GCP](https://cloud.google.com/docs/terraform)
- [Deployment Manager](https://cloud.google.com/deployment-manager/docs)
- [IaC Best Practices](https://cloud.google.com/architecture/devops/devops-tech-infrastructure-as-code)
