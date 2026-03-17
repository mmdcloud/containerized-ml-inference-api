# 🚀 Containerized ML Inference API on AWS

> A production-ready, fully containerized Machine Learning inference solution on AWS — infrastructure automated end-to-end with Terraform.

![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

---

## 📖 Overview

This project delivers a scalable, low-latency ML inference API packaged inside a Docker container and deployed on AWS. The entire cloud infrastructure — from networking to container orchestration to API exposure — is provisioned and managed using Terraform, making it fully reproducible across environments.

Whether you're serving a classification model, a regression model, or a custom deep learning pipeline, this solution gives you a clean, battle-tested template to go from trained model to live REST API endpoint with minimal operational overhead.

---

## 🏗️ Architecture

```
Client Request
      │
      ▼
 API Gateway  ──────────────────────────────────────┐
      │                                              │
      ▼                                              │
 Load Balancer                                       │
      │                                              │
      ▼                                         CloudWatch
 ECS / Lambda (Containerized Inference)         (Logs & Metrics)
      │
      ▼
 ECR (Docker Image Registry)
      │
      ▼
 S3 (Model Artifacts)
```

**Key AWS services used:**
- **Amazon ECR** — Stores the Docker image containing the inference logic
- **Amazon ECS / Lambda** — Runs the containerized inference workload
- **Amazon API Gateway** — Exposes the inference endpoint over HTTPS
- **Amazon S3** — Stores model artifacts
- **Amazon CloudWatch** — Logging, metrics, and alerting
- **IAM** — Fine-grained roles and policies for least-privilege access

---

## 📁 Project Structure

```
containerized-ml-inference-api/
├── infra/                        # Terraform infrastructure code
│   ├── main.tf                   # Root module — orchestrates all resources
│   ├── variables.tf              # Input variable declarations
│   ├── outputs.tf                # Output values (endpoint URLs, ARNs etc.)
│   ├── provider.tf               # AWS provider configuration
│   └── modules/                  # Reusable Terraform modules
│       ├── ecr/                  # ECR repository
│       ├── ecs/                  # ECS cluster and task definitions
│       ├── api_gateway/          # API Gateway configuration
│       ├── iam/                  # IAM roles and policies
│       └── networking/           # VPC, subnets, security groups
├── src/                          # ML inference application source code
│   ├── app.py                    # FastAPI / Flask inference API
│   ├── model.py                  # Model loading and prediction logic
│   ├── requirements.txt          # Python dependencies
│   └── Dockerfile                # Container definition
├── .gitignore
└── README.md
```

---

## ✅ Prerequisites

Before getting started, make sure you have the following installed and configured:

| Tool | Minimum Version | Purpose |
|---|---|---|
| [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) | v2.x | AWS authentication and resource management |
| [Terraform](https://developer.hashicorp.com/terraform/downloads) | v1.3+ | Infrastructure provisioning |
| [Docker](https://docs.docker.com/get-docker/) | v20+ | Building and pushing container images |
| [Python](https://www.python.org/downloads/) | v3.9+ | Running the inference application locally |

**AWS Permissions Required:**

Your AWS IAM user or role must have permissions for: ECR, ECS, API Gateway, S3, IAM, CloudWatch, and VPC.

---

## ⚙️ Setup & Deployment

### 1. Clone the Repository

```bash
git clone https://github.com/mmdcloud/containerized-ml-inference-api.git
cd containerized-ml-inference-api
```

### 2. Configure AWS Credentials

```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and preferred region
```

Or if using a named profile:

```bash
export AWS_PROFILE=your-profile-name
export AWS_REGION=us-east-1
```

### 3. Provision Infrastructure with Terraform

```bash
cd infra

# Initialize Terraform and download providers
terraform init

# Preview what will be created
terraform plan

# Apply the infrastructure
terraform apply
```

> Terraform will output the ECR repository URI and API endpoint URL upon completion. Save these for the next steps.

### 4. Build and Push the Docker Image

```bash
cd ../src

# Authenticate Docker to ECR
aws ecr get-login-password --region $AWS_REGION | \
  docker login --username AWS --password-stdin <your-ecr-uri>

# Build the image
docker build -t ml-inference-api .

# Tag and push
docker tag ml-inference-api:latest <your-ecr-uri>:latest
docker push <your-ecr-uri>:latest
```

### 5. Test the Inference Endpoint

Once deployed, test the live API using `curl`:

```bash
curl -X POST https://<your-api-gateway-url>/predict \
  -H "Content-Type: application/json" \
  -d '{"feature_1": 5.1, "feature_2": 3.5, "feature_3": 1.4, "feature_4": 0.2}'
```

**Expected response:**

```json
{
  "prediction": "setosa",
  "probability": 0.9982,
  "model_version": "1.0.0"
}
```

---

## 🧪 Running Locally

To test the inference API locally before deploying:

```bash
cd src

# Install dependencies
pip install -r requirements.txt

# Run the API server
python app.py

# Test locally
curl -X POST http://localhost:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"feature_1": 5.1, "feature_2": 3.5}'
```

Or using Docker locally:

```bash
docker build -t ml-inference-api .
docker run -p 8080:8080 ml-inference-api
```

---

## 🔧 Configuration

Key variables can be customized in `infra/variables.tf` or passed at apply time:

| Variable | Default | Description |
|---|---|---|
| `aws_region` | `us-east-1` | AWS region to deploy into |
| `project_name` | `ml-inference-api` | Prefix for all named resources |
| `environment` | `dev` | Deployment environment (`dev`, `staging`, `prod`) |
| `container_port` | `8080` | Port exposed by the inference container |
| `task_cpu` | `512` | CPU units allocated to the ECS task |
| `task_memory` | `1024` | Memory (MB) allocated to the ECS task |

Override at apply time:

```bash
terraform apply \
  -var="environment=prod" \
  -var="task_cpu=1024" \
  -var="task_memory=2048"
```

---

## 🔒 Security

This project is built with security best practices in mind:

- **Least-privilege IAM** — Each service has its own scoped IAM role; no wildcard permissions
- **Private ECR** — Container images are stored in a private, encrypted ECR repository
- **VPC isolation** — Inference containers run inside a private VPC subnet
- **HTTPS only** — API Gateway enforces TLS on all incoming requests
- **No hardcoded secrets** — All sensitive values use environment variables or AWS Secrets Manager

---

## 📊 Monitoring & Observability

CloudWatch is configured out of the box:

- **Container logs** — All stdout/stderr from the inference container is streamed to CloudWatch Logs
- **API Gateway metrics** — Request count, latency, and error rates are tracked
- **ECS metrics** — CPU and memory utilization per task

To view logs:

```bash
aws logs tail /ecs/ml-inference-api --follow
```

---

## 🧹 Teardown

To destroy all provisioned AWS resources and avoid ongoing charges:

```bash
cd infra
terraform destroy
```

> **Note:** Manually delete any S3 bucket contents and ECR images before running `terraform destroy`, as Terraform cannot destroy non-empty buckets or registries by default.

---

## 🛠️ Troubleshooting

**`terraform apply` fails with permissions error**
→ Verify your IAM user has the required policies attached. Run `aws sts get-caller-identity` to confirm the correct credentials are active.

**Docker push to ECR fails**
→ Ensure you've run the `aws ecr get-login-password` authentication step and that the ECR repository was successfully created by Terraform first.

**API returns 502 Bad Gateway**
→ The container may still be starting up. Check ECS task status and CloudWatch logs for errors in the application layer.

**Inference returns unexpected results**
→ Confirm the model artifact in S3 matches the version the container expects. Check the `MODEL_PATH` environment variable in the task definition.

---

## 🤝 Contributing

Contributions are welcome! Here's how to get started:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'Add some feature'`
4. Push to your branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

Please make sure Terraform code passes `terraform fmt` and `terraform validate` before submitting.

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**mmdcloud** — [GitHub Profile](https://github.com/mmdcloud)

---

*If this project helped you, consider giving it a ⭐ on GitHub!*
