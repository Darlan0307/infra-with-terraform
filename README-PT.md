# Infra-Caption-Generator

> You can also read in **[English](./README.md)** 🇺🇸

Este repositório é responsável pela **automação da infraestrutura** do projeto [Caption Generator](https://github.com/Darlan0307/Capition-Generate-API) na **AWS**, utilizando **Terraform (IaC)** e **Github Actions (CI/CD)**.

---

## Recursos Criados Automaticamente

Atualmente, os seguintes recursos são provisionados automaticamente via Terraform:

- **ECR Repository**

  - `ecr-caption-generator` para armazenar imagens Docker.

- **EC2 Instance**

  - Instância `t2.micro`.
  - propriedade **user_data** para rodar um arquivo **sh** que instala o Docker na instância quando ela é iniciada.
  - Tags de identificação (`Provisioned = Terraform`, `Cliente = Darlan`).

- **Security Group (website-sg)**
  - Regras configuradas para:
    - **SSH (22)**: acesso restrito ao IP definido na variável `my_ip`.
    - **HTTP (80)**: acesso público.
    - **HTTPS (443)**: acesso público.
    - **Egress (0.0.0.0/0)**: saída liberada para todo tráfego.

---

## Recursos Criados Manualmente (ainda em estudo para automatização)

Alguns recursos foram configurados manualmente, mas serão futuramente incorporados no Terraform:

- **S3 Bucket** (`terraform-state-darlan`) → usado como backend remoto para armazenar o `terraform.tfstate`.
- **Key Pair** (`chave-site`) → necessário para acesso SSH à instância.
- **IAM Role/Instance Profile** (`ECR-EC2-Role`) → permite que a EC2 faça pull de imagens do ECR.
- **VPC** (utilizada no Security Group).

---

## Variáveis Importantes

- **`my_ip`**: define o IP público autorizado para SSH.
  - Configurado no GitHub Secrets como `MY_IP` e passado no workflow.

---

## CI/CD da Infraestrutura

O provisionamento da infraestrutura é automatizado através de um pipeline de **Github Actions**, que executa o ciclo de vida do Terraform.

### Workflow: `Terraform CI/CD`

- **Disparo**: manual via `workflow_dispatch`.
- **Inputs Disponíveis**:
  - `apply`: executa `terraform apply` para criar/atualizar recursos.
  - `destroy`: executa `terraform destroy` para remover os recursos.
  - `plan_destroy`: apenas planeja a destruição (`terraform plan -destroy`).

### Etapas do Job

1. **Checkout** do código.
2. **Configuração de credenciais AWS** (assume role configurada via `ROLE_ARN_INFRA`).
3. **Setup Terraform**.
4. **Terraform Init** (inicialização com backend remoto no S3).
5. **Terraform Validate** (validação da configuração).
6. **Terraform Plan** (gera plano de execução).
7. **Terraform Apply** (executa alterações no ambiente).
8. **Plan Destroy** (plano de destruição opcional).
9. **Terraform Destroy** (destruição total do ambiente).
