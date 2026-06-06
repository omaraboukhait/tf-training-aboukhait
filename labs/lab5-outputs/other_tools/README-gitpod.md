# 📘 Lab 5 — Outputs & Variables (GitPod)

## 🎯 Objectifs du Lab

- Exporter des attributs avec les **outputs**
- Déclarer et utiliser des **variables** avec validation
- Assigner via **CLI**, **tfvars** et **TF_VAR_**

---

# 🧩 Étape 1 — Préparer l'environnement

Reprenez votre environnement GitPod → **https://gitpod.io/workspaces**

```bash
cd labs/lab5-outputs
```

Créez les fichiers dans l'éditeur VS Code GitPod :

### `versions.tf`

```hcl
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
  backend "s3" {
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }
}
provider "aws" { region = var.region }
```

### `variables.tf`

```hcl
variable "username" {
  description = "Votre prenom"
  type        = string
  validation {
    condition     = length(var.username) >= 2 && length(var.username) <= 20
    error_message = "Le username doit contenir entre 2 et 20 caractères."
  }
}
variable "region" {
  description = "Region AWS"
  type        = string
  default     = "eu-west-1"
}
variable "instance_type" {
  description = "Type d'instance EC2"
  type        = string
  default     = "t2.micro"
  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Le type d'instance doit être t2.micro, t2.small ou t2.medium."
  }
}
variable "environment" {
  description = "Environnement"
  type        = string
  default     = "dev"
  validation {
    condition     = contains(["dev", "prod"], var.environment)
    error_message = "L'environnement doit être dev ou prod."
  }
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab5_sg" {
  name        = "lab5-sg-${var.username}-${var.environment}"
  description = "Security group Lab 5 - ${var.username}"
  tags = { Name = "lab5-sg-${var.username}", Lab = "lab5", Username = var.username, Environment = var.environment }
}

resource "aws_instance" "lab5_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = var.instance_type
  vpc_security_group_ids = [aws_security_group.lab5_sg.id]
  tags = { Name = "lab5-instance-${var.username}", Lab = "lab5", Username = var.username, Environment = var.environment }
}
```

### `outputs.tf`

```hcl
output "instance_id" { value = aws_instance.lab5_instance.id }
output "instance_public_ip" { value = aws_instance.lab5_instance.public_ip }
output "security_group_id" { value = aws_security_group.lab5_sg.id }
output "security_group_name" { value = aws_security_group.lab5_sg.name }
output "deployment_summary" {
  value = { username = var.username, environment = var.environment, instance = aws_instance.lab5_instance.id, region = var.region }
}
```

---

# 🧩 Étape 2 — Init avec Backend S3

```bash
BUCKET_NAME=$(grep "bucket" $HOME/tf-training-info.txt | awk '{print $NF}')

terraform init \
  -backend-config="bucket=${BUCKET_NAME}" \
  -backend-config="key=terraform.tfstate"
```

---

# 🧩 Étape 3 — Les 3 façons d'assigner les variables

## Méthode 1 — CLI flag

```bash
terraform apply -var="username=<votre-prenom>" -var="environment=dev"
```

## Méthode 2 — tfvars

Créez `terraform.tfvars` dans l'éditeur VS Code GitPod :

```hcl
username      = "<votre-prenom>"
environment   = "dev"
instance_type = "t2.micro"
```

```bash
terraform apply
```

## Méthode 3 — TF_VAR_

```bash
export TF_VAR_username="<votre-prenom>"
export TF_VAR_environment="dev"
terraform apply
```

---

# 🧩 Étape 4 — Tester la validation

```bash
terraform plan -var="username=<votre-prenom>" -var="environment=staging"
terraform plan -var="username=<votre-prenom>" -var="instance_type=t3.large"
```

---

# 🧩 Étape 5 — Utiliser terraform output

```bash
terraform output
terraform output instance_id
terraform output -json deployment_summary
terraform output -raw instance_id
```

---

# 🧩 Étape 6 — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>"
```

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | Validation bloque `environment=staging` | ☐ |
| 2 | Validation bloque `instance_type=t3.large` | ☐ |
| 3 | Les 3 méthodes d'assignation fonctionnent | ☐ |
| 4 | `terraform output -json` affiche un objet JSON | ☐ |
| 5 | `terraform destroy` supprime les ressources | ☐ |

---

🎉 **Fin du Lab 5 !**
