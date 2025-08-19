
# **GCP Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [gcloud CLI Configuration](#gcloud-cli-configuration)
3. [Projects & IAM](#projects--iam)
4. [Compute Engine (VMs)](#compute-engine-vms)
5. [Cloud Storage](#cloud-storage)
6. [Networking](#networking)
7. [Cloud Functions](#cloud-functions)
8. [Monitoring & Logging](#monitoring--logging)
9. [Other Useful Commands](#other-useful-commands)
10. [Best Practices](#best-practices)

### **Overview**

* `gcloud` CLI manages Google Cloud resources.
* Services: **Compute Engine, Cloud Storage, Cloud Functions, Networking**.

### **gcloud CLI Configuration**

```bash
# Initialize gcloud
gcloud init

# Set default project
gcloud config set project my-project-id

# List configurations
gcloud config list

# Show account info
gcloud auth list
```

### **Projects & IAM**

```bash
# List projects
gcloud projects list

# Create project
gcloud projects create my-project-id --name "My Project"

# Add IAM role
gcloud projects add-iam-policy-binding my-project-id --member="user:john.doe@example.com" --role="roles/editor"

# Remove IAM role
gcloud projects remove-iam-policy-binding my-project-id --member="user:john.doe@example.com" --role="roles/editor"
```

### **Compute Engine (VMs)**

```bash
# List VMs
gcloud compute instances list

# Create VM
gcloud compute instances create my-vm --zone=us-central1-a --machine-type=e2-medium --image-family=debian-11 --image-project=debian-cloud

# Start VM
gcloud compute instances start my-vm --zone=us-central1-a

# Stop VM
gcloud compute instances stop my-vm --zone=us-central1-a

# Delete VM
gcloud compute instances delete my-vm --zone=us-central1-a
```

### **Cloud Storage**

```bash
# List buckets
gcloud storage buckets list

# Create bucket
gcloud storage buckets create gs://my-bucket --location=us-central1

# Upload file
gcloud storage cp file.txt gs://my-bucket/

# Download file
gcloud storage cp gs://my-bucket/file.txt ./file.txt

# List objects
gcloud storage ls gs://my-bucket
```

### **Networking**

```bash
# List VPC networks
gcloud compute networks list

# Create VPC network
gcloud compute networks create my-vpc --subnet-mode=custom

# List firewall rules
gcloud compute firewall-rules list

# Create firewall rule
gcloud compute firewall-rules create allow-ssh --allow tcp:22 --network=my-vpc
```

### **Cloud Functions**

```bash
# List functions
gcloud functions list

# Deploy function
gcloud functions deploy my-function --runtime python310 --trigger-http --allow-unauthenticated

# Call function
gcloud functions call my-function --data '{"key":"value"}'

# Delete function
gcloud functions delete my-function
```
