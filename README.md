# Infra-Caption-Generator

> Você também pode ler em **[Português](./README-PT.md)** 🇧🇷

This repository is responsible for the **infrastructure automation** of the project [Caption Generator](https://github.com/Darlan0307/Capition-Generate-API) on **AWS**, using **Terraform (IaC)** and **GitHub Actions (CI/CD)**.

---

## Automatically Created Resources

Currently, the following resources are provisioned automatically via Terraform:

- **ECR Repository**

  - `ecr-caption-generator` to store Docker images.

- **EC2 Instance**

  - `t2.micro` instance.
  - **user_data** property to run a **sh** script that installs Docker on the instance when it starts.
  - Identification tags (`Provisioned = Terraform`, `Cliente = Darlan`).

- **Security Group (website-sg)**
  - Configured rules:
    - **SSH (22)**: restricted access to the IP defined in the `my_ip` variable.
    - **HTTP (80)**: public access.
    - **HTTPS (443)**: public access.
    - **Egress (0.0.0.0/0)**: all outbound traffic allowed.

---

## Manually Created Resources (under study for automation)

Some resources were configured manually but will be incorporated into Terraform in the future:

- **S3 Bucket** (`terraform-state-darlan`) → used as a remote backend to store the `terraform.tfstate`.
- **Key Pair** (`chave-site`) → required for SSH access to the instance.
- **IAM Role/Instance Profile** (`ECR-EC2-Role`) → allows EC2 to pull images from ECR.
- **VPC** (used by the Security Group).

---

## Important Variables

- **`my_ip`**: defines the public IP authorized for SSH access.
  - Configured in GitHub Secrets as `MY_IP` and passed into the workflow.

---

## Infrastructure CI/CD

Infrastructure provisioning is automated through a **GitHub Actions** pipeline that executes the Terraform lifecycle.

### Workflow: `Terraform CI/CD`

- **Trigger**: manual via `workflow_dispatch`.
- **Available Inputs**:
  - `apply`: runs `terraform apply` to create/update resources.
  - `destroy`: runs `terraform destroy` to remove resources.
  - `plan_destroy`: only plans the destruction (`terraform plan -destroy`).

### Job Steps

1. **Checkout** the code.
2. **AWS credentials configuration** (assumes role configured via `ROLE_ARN_INFRA`).
3. **Setup Terraform**.
4. **Terraform Init** (initialization with remote backend on S3).
5. **Terraform Validate** (validate the configuration).
6. **Terraform Plan** (generate execution plan).
7. **Terraform Apply** (apply changes to the environment).
8. **Plan Destroy** (optional destruction plan).
9. **Terraform Destroy** (complete environment destruction).
