# Challenge

## Task 1: Deploy EKS NodeGroup CloudFormation Template\*\*

### **Prerequisites**

1. **EKS Cluster**:

   - Note the cluster name.

2. **IAM Role**:

   - The EKS NodeGroup requires an IAM role with the necessary permissions:
     - `AmazonEKSWorkerNodePolicy`
     - `AmazonEC2ContainerRegistryReadOnly`
     - `AmazonEKS_CNI_Policy`
   - Ensure the role's ARN is exported (if using a separate upstream stack) or included in the template.

3. **Subnets**:
   - Identify the subnet IDs in different Availability Zones where the NodeGroup will deploy.

### **Deployment Using AWS Management Console**

1. Open the [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation/).
2. Click **Create stack** > **With new resources (standard)**.
3. Upload the CloudFormation template file (`eks-nodegroup.yaml`).
4. Provide the required parameters:
   - **ClusterName**: Name of your existing EKS cluster.
   - **SubnetIds**: List of subnet IDs (comma-separated).
5. Click **Next** and configure stack options (e.g., tags, permissions).
6. Click **Next**, review the settings, and click **Create stack**.
7. Wait for the stack to reach the **CREATE_COMPLETE** status.

---

## Task 2: Deploying Nginx on EKS Using Helm

This guide explains how to deploy an Nginx container on Amazon EKS using a Helm chart. The process includes setting up the kubeconfig for EKS, installing Helm, and deploying the chart.

### Prerequisites

- AWS CLI installed and configured
- kubectl installed
- Helm installed
- An EKS cluster already created

### Steps to Deploy

### Step 1: Configure kubeconfig for EKS

Run the following command to update your kubeconfig file to use your EKS cluster:

```bash
aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
```

Replace `<your-region>` with your AWS region (e.g., `us-west-2`) and `<your-cluster-name>` with the name of your EKS cluster.

To verify the configuration, run:

```bash
kubectl get nodes
```

You should see a list of nodes in your EKS cluster.

### Step 2: Install Helm

If Helm is not installed, follow these steps:

1. Download and install Helm:

   ```bash
   curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```

2. Verify the installation:

   ```bash
   helm version
   ```

### Step 3: Clone or Copy the Helm Chart

Ensure the `nginx-helm-chart` directory is ready in your working directory. It should contain:

- `Chart.yaml`
- `values.yaml`
- `templates/`

### Step 4: Deploy the Helm Chart

1. Deploy the Helm chart to your EKS cluster:

   ```bash
   helm install my-nginx ./nginx-helm-chart
   ```

   Here, `my-nginx` is the release name. You can replace it with a name of your choice.

2. Verify the deployment:

   ```bash
   kubectl get pods
   kubectl get svc
   ```

   You should see the pods running and the service exposing an external IP.

### Step 5: Access the Nginx Service

1. Identify the external IP of the LoadBalancer service:

   ```bash
   kubectl get svc
   ```

   Look for the `EXTERNAL-IP` column in the output.

2. Open the external IP in your web browser to access the Nginx service.

### Cleanup

To delete the deployment and free resources:

```bash
helm uninstall my-nginx
```

---

## Task 3 CI/CD Pipeline for Building and Pushing Docker Images to ECR

This repository includes a GitHub Actions workflow to automate the process of building a Docker image from the `example-code` folder and pushing it to a private Amazon Elastic Container Registry (ECR).

### Workflow Overview

The CI/CD pipeline is defined in the `.github/workflows/docker-ecr.yml` file. It is triggered when changes are pushed to the `main` branch (or any other branch specified in the workflow).

### Pipeline Steps

1. **Checkout Code**:

   - The pipeline uses the [`actions/checkout`](https://github.com/actions/checkout) action to pull the repository code into the workflow environment.

2. **Authenticate with ECR**:

   - The [`aws-actions/amazon-ecr-login`](https://github.com/aws-actions/amazon-ecr-login) action authenticates with Amazon ECR using the AWS credentials provided as secrets.

3. **Build Docker Image**:

   - The Docker image is built from the `example-code` directory, using the `Dockerfile` located there.

4. **Push Docker Image to ECR**:
   - The built image is tagged with the ECR repository URL and pushed to the private ECR repository.

### Secrets Configuration

The pipeline uses GitHub Secrets to securely store and retrieve sensitive information. The following secrets must be added to the repository:

- **`AWS_ACCESS_KEY_ID`**: AWS Access Key for the IAM user with ECR permissions.
- **`AWS_SECRET_ACCESS_KEY`**: AWS Secret Key for the IAM user.
- **`AWS_REGION`**: AWS region where the ECR repository is hosted (e.g., `us-west-2`).

### Prerequisites

1. **AWS Setup**:

   - Ensure an Amazon ECR repository is created in your AWS account.
   - The IAM user or role must have the following permissions:
     - `ecr:GetAuthorizationToken`
     - `ecr:BatchCheckLayerAvailability`
     - `ecr:InitiateLayerUpload`
     - `ecr:UploadLayerPart`
     - `ecr:CompleteLayerUpload`
     - `ecr:PutImage`

2. **GitHub Repository Setup**:

   - Add the AWS secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_REGION`) in your repository settings.

3. **Dockerfile**:
   - Ensure the `example-code` folder contains a valid `Dockerfile` for building your application.

### Triggering the Workflow

The pipeline runs automatically when:

- Code is pushed to the `main` branch (or any branch specified in the workflow).

You can also trigger the workflow manually from the **Actions** tab in your GitHub repository.

### How It Works

1. A developer makes changes to the application code in the `example-code` folder and pushes them to the `main` branch.

2. The GitHub Actions workflow is triggered automatically.

3. The pipeline performs the following actions:

   - Checks out the code.
   - Authenticates with Amazon ECR using the credentials stored in GitHub Secrets.
   - Builds a Docker image from the `example-code` folder.
   - Tags the image with the ECR registry URL and repository name.
   - Pushes the image to the private ECR repository.

4. The updated image is now available in ECR for deployment.

### Debugging

If the workflow fails:

- Check the logs in the **Actions** tab of your GitHub repository.
- Ensure the secrets are correctly configured.
- Verify that the ECR repository exists and the IAM user has the correct permissions.
