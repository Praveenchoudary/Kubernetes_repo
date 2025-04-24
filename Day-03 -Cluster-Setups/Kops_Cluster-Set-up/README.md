# Kops Cluster Setup on AWS - praveens.online

This guide provides a step-by-step process to set up a Kubernetes cluster using **Kops** on AWS with the domain **praveens.online**. Follow these instructions to create and configure your Kubernetes cluster on AWS.

---

## Prerequisites

Ensure you have the following tools installed:

- **AWS CLI**: To interact with AWS services.
- **kubectl**: To manage Kubernetes clusters.
- **Kops**: To create and manage Kubernetes clusters on AWS.

### Installation Steps:

1. **Install AWS CLI**:
   ```bash
   sudo apt update
   sudo apt install -y awscli

    Install kubectl:

sudo apt install -y kubectl

Install Kops:

    curl -LO https://github.com/kubernetes/kops/releases/download/v1.28.0/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops

Step 1: Configure AWS CLI

Run the following command to configure AWS CLI:

aws configure

You will be prompted to enter your AWS Access Key ID, AWS Secret Access Key, Region, and Output format (e.g., json).
Step 2: Set Up DNS with Route 53

    Log in to the AWS Console and navigate to Route 53.

    Create a Public Hosted Zone for praveens.online.

    Update your domain registrar (e.g., GoDaddy) with the name servers provided by AWS Route 53.

Step 3: Create an S3 Bucket for Kops State Store

Kops requires an S3 bucket to store the cluster state. Run the following commands to create the bucket and enable versioning:

aws s3api create-bucket --bucket kops-state-praveens-online --region us-east-1
aws s3api put-bucket-versioning --bucket kops-state-praveens-online --versioning-configuration Status=Enabled

Step 4: Generate SSH Key Pair

Generate an SSH key pair to access the Kubernetes nodes:

ssh-keygen -t rsa -b 4096

Press Enter to accept the default file location and create the SSH key. This key will be used for node access.
Step 5: Set Environment Variables

Set environment variables to simplify the process. Run the following commands:

export KOPS_STATE_STORE=s3://kops-state-praveens-online
export NAME=praveens.online

    KOPS_STATE_STORE: Specifies where the cluster state is stored (S3 bucket).

    NAME: Your domain name (praveens.online).

Step 6: Create the Cluster

Use kops to create the cluster. Run this command:

kops create cluster \
  --name=${NAME} \
  --cloud=aws \
  --zones=us-east-1a \
  --dns-zone=${NAME} \
  --master-size=t3.medium \
  --node-size=t3.medium \
  --node-count=2 \
  --ssh-public-key=~/.ssh/id_rsa.pub

Explanation of parameters:

    --name=${NAME}: The name of the cluster, based on your domain name (praveens.online).

    --cloud=aws: Specifies the cloud provider (AWS).

    --zones=us-east-1a: Availability zones for the cluster.

    --dns-zone=${NAME}: The Route 53 DNS zone for your domain.

    --master-size=t3.medium: EC2 instance type for the Kubernetes master node.

    --node-size=t3.medium: EC2 instance type for the worker nodes.

    --node-count=2: Number of worker nodes.

    --ssh-public-key=~/.ssh/id_rsa.pub: Path to your SSH public key for accessing the nodes.

Step 7: Apply the Cluster Configuration

Run the following command to apply the cluster configuration and start provisioning:

kops update cluster --name=${NAME} --yes

The --yes flag automatically applies the configuration.
Step 8: Validate the Cluster

To ensure the cluster is set up correctly, run the following command:

kops validate cluster --name=${NAME}

This command checks whether the cluster is valid, and all nodes are running.
