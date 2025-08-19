# **AWS Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [AWS CLI Configuration](#aws-cli-configuration)
3. [S3 (Simple Storage Service)](#s3-simple-storage-service)

   * [Bucket Management](#bucket-management)
   * [Objects Management](#objects-management)
   * [Advanced S3 Commands](#advanced-s3-commands)
4. [EC2 (Elastic Compute Cloud)](#ec2-elastic-compute-cloud)

   * [Instance Management](#instance-management)
   * [Security Groups & Key Pairs](#security-groups--key-pairs)
   * [Elastic IPs & Volumes](#elastic-ips--volumes)
5. [IAM (Identity & Access Management)](#iam-identity--access-management)

   * [User & Group Management](#user--group-management)
   * [Roles & Policies](#roles--policies)
6. [Lambda (Serverless Functions)](#lambda-serverless-functions)
7. [CloudWatch](#cloudwatch)
8. [VPC (Networking)](#vpc-networking)
9. [Other Useful CLI Commands](#other-useful-cli-commands)
10. [Best Practices](#best-practices)

---

## **Overview**

* AWS CLI allows **full AWS resource management** from terminal.
* Key services: **S3, EC2, IAM, Lambda, CloudWatch, VPC**.
* Supports automation with scripts.

---

## **AWS CLI Configuration**

```bash
aws configure                    # Set credentials, region, output
aws configure list               # Show current config
aws configure get region         # Show default region
aws configure list-profiles      # List CLI profiles
aws s3 ls --profile myprofile    # Use specific profile
```

---

## **S3 (Simple Storage Service)**

### **Bucket Management**

```bash
aws s3 ls                           # List buckets
aws s3 mb s3://my-bucket            # Create bucket
aws s3 rb s3://my-bucket            # Delete bucket
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled
aws s3api get-bucket-versioning --bucket my-bucket
aws s3api get-bucket-acl --bucket my-bucket
```

### **Objects Management**

```bash
aws s3 cp file.txt s3://my-bucket/                  # Upload
aws s3 cp s3://my-bucket/file.txt ./               # Download
aws s3 cp /local/dir s3://my-bucket/ --recursive  # Upload dir
aws s3 sync /local/dir s3://my-bucket/            # Sync
aws s3 ls s3://my-bucket/                          # List objects
aws s3 rm s3://my-bucket/file.txt                  # Delete object
aws s3 rm s3://my-bucket/ --recursive             # Delete all
```

### **Advanced S3 Commands**

```bash
aws s3api put-bucket-lifecycle-configuration --bucket my-bucket --lifecycle-configuration file://lifecycle.json
aws s3api get-bucket-lifecycle-configuration --bucket my-bucket
aws s3api put-bucket-encryption --bucket my-bucket --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

---

## **EC2 (Elastic Compute Cloud)**

### **Instance Management**

```bash
aws ec2 describe-instances
aws ec2 run-instances --image-id ami-0123456789abcdef0 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-0123456789abcdef0 --subnet-id subnet-0123456789abcdef0
aws ec2 start-instances --instance-ids i-0123456789abcdef0
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
aws ec2 reboot-instances --instance-ids i-0123456789abcdef0
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

### **Security Groups & Key Pairs**

```bash
aws ec2 create-security-group --group-name MySG --description "My security group"
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 revoke-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
chmod 400 MyKeyPair.pem
```

### **Elastic IPs & Volumes**

```bash
aws ec2 allocate-address                     # Allocate Elastic IP
aws ec2 associate-address --instance-id i-0123456789abcdef0 --allocation-id eipalloc-0123456789abcdef0
aws ec2 release-address --allocation-id eipalloc-0123456789abcdef0

aws ec2 create-volume --size 10 --availability-zone us-east-1a
aws ec2 attach-volume --volume-id vol-0123456789abcdef0 --instance-id i-0123456789abcdef0 --device /dev/sdf
aws ec2 detach-volume --volume-id vol-0123456789abcdef0
aws ec2 delete-volume --volume-id vol-0123456789abcdef0
```

---

## **IAM (Identity & Access Management)**

### **User & Group Management**

```bash
aws iam create-user --user-name JohnDoe
aws iam list-users
aws iam create-group --group-name Admins
aws iam add-user-to-group --user-name JohnDoe --group-name Admins
aws iam remove-user-from-group --user-name JohnDoe --group-name Admins
aws iam attach-user-policy --user-name JohnDoe --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam detach-user-policy --user-name JohnDoe --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

### **Roles & Policies**

```bash
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust-policy.json
aws iam list-roles
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam detach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

---

## **Lambda (Serverless Functions)**

```bash
aws lambda list-functions
aws lambda create-function --function-name MyFunc --runtime python3.11 --role arn:aws:iam::123456789012:role/MyRole --handler lambda_function.lambda_handler --zip-file fileb://function.zip
aws lambda invoke --function-name MyFunc output.json
aws lambda update-function-code --function-name MyFunc --zip-file fileb://function.zip
aws lambda delete-function --function-name MyFunc
```

---

## **CloudWatch**

```bash
aws logs describe-log-groups
aws logs create-log-group --log-group-name MyLogGroup
aws logs create-log-stream --log-group-name MyLogGroup --log-stream-name MyStream
aws logs put-log-events --log-group-name MyLogGroup --log-stream-name MyStream --log-events file://events.json
aws cloudwatch list-metrics
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --start-time 2025-08-20T00:00:00Z --end-time 2025-08-20T23:59:59Z --period 300 --statistics Average --dimensions Name=InstanceId,Value=i-0123456789abcdef0
```

---

## **VPC (Networking)**

```bash
aws ec2 describe-vpcs
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 describe-subnets
aws ec2 create-subnet --vpc-id vpc-0123456789abcdef0 --cidr-block 10.0.1.0/24
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-0123456789abcdef0 --internet-gateway-id igw-0123456789abcdef0
aws ec2 create-route-table --vpc-id vpc-0123456789abcdef0
```

---

## **Other Useful CLI Commands**

```bash
aws ec2 describe-regions
aws ec2 describe-availability-zones
aws pricing get-products --service-code AmazonEC2 --region us-east-1
aws sts get-caller-identity       # Verify current IAM identity
aws s3 presign s3://my-bucket/file.txt --expires-in 3600
aws cloudformation list-stacks
aws cloudformation describe-stack-resources --stack-name my-stack
```

---

## **Best Practices**

1. Use **IAM roles instead of hardcoding credentials**.
2. Enable **S3 versioning** and **server-side encryption**.
3. Limit **security group access** to necessary IPs/ports.
4. Use **AWS CLI profiles** for multiple accounts.
5. Rotate IAM access keys regularly.
6. Automate with scripts and **CloudWatch events**.
7. Monitor **Lambda, CloudWatch, and billing alerts**.
8. Tag resources for **tracking and cost management**.

