# 📘 Terraform: Infrastructure as Code (IaC) – A Comprehensive Guide

---

## 🧩 What is Terraform?

**Terraform** is an open-source **Infrastructure as Code (IaC)** tool developed by **HashiCorp** that allows you to define, provision, and manage cloud infrastructure using a declarative configuration language known as **HCL (HashiCorp Configuration Language)**.

Instead of manually creating AWS resources (like EC2 instances, S3 buckets, Lambda functions, etc.) through the AWS console, Terraform allows you to write `.tf` files and version control your infrastructure just like application code.

### ✅ Key Features:

- **Cloud-Agnostic**: Supports AWS, Azure, GCP, Oracle, and more.
- **Declarative Language**: Define desired state; Terraform figures out how to achieve it.
- **Dependency Management**: Handles dependencies between resources automatically.
- **Modular**: Promote reuse with modules.
- **State Management**: Keeps track of infrastructure in a `terraform.tfstate` file.

---

## 🔁 Terraform vs. AWS SAM vs. Serverless Framework

| Feature / Tool              | Terraform                                  | AWS SAM                                      | Serverless Framework                           |
|----------------------------|--------------------------------------------|----------------------------------------------|-------------------------------------------------|
| **Type**                   | Infrastructure as Code (IaC)               | Serverless-specific IaC                      | Serverless Framework (multi-cloud, plugin-rich) |
| **Language**               | HCL (Hashicorp Configuration Language)                                       | YAML                                         | YAML + JavaScript plugins                       |
| **Scope**                  | Broad: EC2, S3, RDS, Lambda, VPC, etc.     | Narrow: AWS Lambda & related services        | Serverless focus (multi-cloud)                  |
| **State Management**       | Yes (remote or local state)                | No built-in; relies on CloudFormation        | No state file (uses CloudFormation for AWS)     |
| **Multi-cloud Support**    | Yes                                        | No                                           | Yes (AWS, Azure, GCP)                           |
| **CLI Experience**         | `terraform apply`, `plan`, etc.            | `sam build`, `sam deploy`                    | `sls deploy`, `sls invoke`                      |
| **Extensibility**          | Modules & providers                        | Limited                                      | Plugin system                                    |

---

## 🔧 Terraform Example: Creating an AWS S3 Bucket and Lambda

```hcl
provider "aws" {
  region = "us-west-1"
}

resource "aws_s3_bucket" "upload_bucket" {
  bucket = "my-upload-bucket-demo"
  acl    = "private"
}

resource "aws_iam_role" "lambda_exec_role" {
  name = "lambda_exec_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action = "sts:AssumeRole",
      Principal = {
        Service = "lambda.amazonaws.com"
      },
      Effect = "Allow",
      Sid = ""
    }]
  })
}

resource "aws_lambda_function" "s3_handler" {
  filename         = "lambda.zip"
  function_name    = "s3LambdaHandler"
  role             = aws_iam_role.lambda_exec_role.arn
  handler          = "index.handler"
  runtime          = "nodejs18.x"
  source_code_hash = filebase64sha256("lambda.zip")
}

resource "aws_lambda_permission" "allow_s3" {
  statement_id  = "AllowExecutionFromS3"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.s3_handler.function_name
  principal     = "s3.amazonaws.com"
}
```

---

## 🧪 Equivalent in AWS SAM

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  S3LambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs18.x
      CodeUri: .
      Events:
        S3Event:
          Type: S3
          Properties:
            Bucket: my-upload-bucket-demo
            Events: s3:ObjectCreated:*
```

---

## 🧪 Equivalent in Serverless Framework

```yaml
service: s3-handler-service

provider:
  name: aws
  runtime: nodejs18.x

functions:
  s3Handler:
    handler: index.handler
    events:
      - s3:
          bucket: my-upload-bucket-demo
          event: s3:ObjectCreated:*
```

---

## ✅ When to Use Which?

| Scenario                                        | Best Tool        |
|------------------------------------------------|------------------|
| Full cloud infrastructure (VPC, EC2, RDS, etc) | **Terraform**    |
| Pure AWS Lambda apps (simpler projects)        | **AWS SAM**      |
| Multi-cloud or plugin-rich serverless app      | **Serverless Framework** |
| CI/CD ready IaC with state tracking            | **Terraform**    |

---

## 📌 Things to Consider When Working with Terraform

### 🔒 State Management

- Use **remote state** (e.g., S3 + DynamoDB) for team collaboration.
- Protect `terraform.tfstate` from version control.

### 🔁 Resource Naming Conventions

- Use environment prefixes (`dev`, `prod`, etc.)
- Use `terraform.workspace` for environment isolation.

### 📦 Module Usage

- Break your infrastructure into **reusable modules**.
- Example: `ec2_instance`, `vpc`, `lambda_function` modules.

### 🔐 Security Best Practices

- Use IAM roles over access keys.
- Encrypt S3 state files with SSE.
- Use `kms_key_id` for sensitive resources.

### ⚠️ Planning Before Applying

- Always run `terraform plan` to see proposed changes.
- Use `-target` flag only in emergencies (can lead to drift).

### ✅ Version Locking

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 🔄 Lifecycle Customization

```hcl
resource "aws_lambda_function" "example" {
  lifecycle {
    create_before_destroy = true
    ignore_changes        = [last_modified]
  }
}
```

---

## 🛠️ Useful Terraform Commands

```bash
terraform init       # Initialize the project
terraform plan       # Preview changes
terraform apply      # Apply changes
terraform destroy    # Destroy infrastructure
terraform fmt        # Format code
terraform validate   # Validate syntax
terraform state list # See all managed resources
```

---

## 🌐 Real-World Terraform Structure

```
infra/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── modules/
│   ├── ec2-instance/
│   └── s3-bucket/
└── environments/
    ├── dev/
    └── prod/
```

---

## 🧠 Pro Tips

- Use **Terragrunt** for DRY, hierarchical config and environment management.
- Use **pre-commit hooks** to run `terraform fmt` and `validate`.
- Document resource links and purpose in comments.
- Use data sources to fetch existing resources safely.
- Use `local-exec`/`null_resource` with care – avoid if possible.
- Tag all resources for cost tracking & auditing.

---

## 🧾 References

- [Terraform Docs](https://developer.hashicorp.com/terraform/docs)
- [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/)
- [Serverless Framework](https://www.serverless.com/framework/docs)

<script>
function copyCode(button) {
  const code = button.nextElementSibling.innerText;
  navigator.clipboard.writeText(code).then(() => {
    button.innerText = "✅ Copied";
    setTimeout(() => (button.innerText = "📋 Copy"), 2000);
  });
}
</script>