# 📘 Lab 6 — Locals (GitPod)

## 🎯 Objectifs du Lab

- Déclarer des **locals** pour centraliser les expressions
- Référencer avec `local.<nom>`
- Utiliser les locals pour les tags communs et les noms de ressources

---

# 🧩 Étape 1 — Créer les fichiers

Reprenez votre environnement GitPod → **https://gitpod.io/workspaces**

```bash
cd labs/lab06-locals
```

### `backend.tf`

```hcl
terraform {
  backend "s3" {
    bucket       = "tf-training-<votre-prenom>-982908300187"
    key          = "terraform.tfstate"
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }
}
```

> ⚠️ Remplacez `<votre-prenom>` par votre prénom en minuscules sans accent. Ex : `tf-training-alice-982908300187`

### `versions.tf`

```hcl
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
```

### `provider.tf`

```hcl
provider "aws" { region = var.region }
```

### `variables.tf`

```hcl
variable "username" { type = string }
variable "region" { type = string; default = "eu-west-1" }
variable "environment" {
  type    = string
  default = "dev"
  validation {
    condition     = contains(["dev", "prod"], var.environment)
    error_message = "L'environnement doit être dev ou prod."
  }
}
```

### `locals.tf`

```hcl
locals {
  prefix        = "lab06-${var.username}-${var.environment}"
  instance_type = var.environment == "prod" ? "t2.small" : "t2.micro"
  common_tags = {
    Lab         = "lab6"
    Username    = var.username
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab6_sg" {
  name        = "${local.prefix}-sg"
  description = "Security group ${local.prefix}"
  tags        = merge(local.common_tags, { Name = "${local.prefix}-sg" })
}

resource "aws_instance" "lab6_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = local.instance_type
  vpc_security_group_ids = [aws_security_group.lab6_sg.id]
  tags                   = merge(local.common_tags, { Name = "${local.prefix}-instance" })
}
```

### `outputs.tf`

```hcl
output "instance_id" { value = aws_instance.lab6_instance.id }
output "instance_type_used" { value = local.instance_type }
output "prefix_used" { value = local.prefix }
```

---

# 🧩 Étape 2 — Déployer et observer

```bash
terraform init
terraform apply -var="username=<votre-prenom>" -var="environment=dev"
terraform output instance_type_used
```

---

# 🧩 Étape 3 — Tester le local conditionnel

```bash
terraform apply -var="username=<votre-prenom>" -var="environment=prod"
terraform output instance_type_used
```

---

# 🧩 Étape 4 — terraform console

```bash
terraform console -var="username=<votre-prenom>" -var="environment=dev"
```

```hcl
> local.prefix
> local.instance_type
> local.common_tags
```

Tapez `exit` pour quitter.

---

# 🧩 Étape 5 — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>" -var="environment=dev"
```

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | `locals.tf` créé avec `prefix`, `common_tags`, `instance_type` | ☐ |
| 2 | `instance_type_used` retourne `t2.micro` en dev | ☐ |
| 3 | `instance_type_used` retourne `t2.small` en prod | ☐ |
| 4 | `terraform console` affiche les valeurs des locals | ☐ |
| 5 | `terraform destroy` supprime les ressources | ☐ |

---

🎉 **Fin du Lab 6 !**
