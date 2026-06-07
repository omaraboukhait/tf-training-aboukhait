# 📘 Lab 7 — Count & Count Index (GitPod)

## 🎯 Objectifs du Lab

- Utiliser **count** pour créer plusieurs instances
- Utiliser **count.index** pour différencier les ressources
- Respecter le principe **DRY**

---

# 🧩 Étape 1 — Créer les fichiers

Reprenez votre environnement GitPod → **https://gitpod.io/workspaces**

```bash
cd labs/lab07-count
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
variable "instance_count" {
  type    = number
  default = 3
  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 5
    error_message = "Le nombre d'instances doit être entre 1 et 5."
  }
}
```

### `locals.tf`

```hcl
locals {
  prefix      = "lab07-${var.username}"
  common_tags = { Lab = "lab7", Username = var.username, ManagedBy = "Terraform" }
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab7_sg" {
  name        = "${local.prefix}-sg"
  description = "Security group Lab 7 - ${var.username}"
  tags        = merge(local.common_tags, { Name = "${local.prefix}-sg" })
}

resource "aws_instance" "lab7_instances" {
  count                  = var.instance_count
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab7_sg.id]
  tags = merge(local.common_tags, {
    Name  = "${local.prefix}-instance-${count.index + 1}"
    Index = count.index
  })
}
```

### `outputs.tf`

```hcl
output "instance_ids"   { value = aws_instance.lab7_instances[*].id }
output "instance_names" { value = aws_instance.lab7_instances[*].tags.Name }
output "instance_count" { value = length(aws_instance.lab7_instances) }
```

---

# 🧩 Étape 2 — Déployer 3 instances

```bash
terraform init
terraform apply -var="username=<votre-prenom>"
terraform output instance_ids
terraform output instance_names
```

---

# 🧩 Étape 3 — Modifier le count

```bash
# Augmenter à 5
terraform apply -var="username=<votre-prenom>" -var="instance_count=5"

# Réduire à 1
terraform apply -var="username=<votre-prenom>" -var="instance_count=1"
```

---

# 🧩 Étape 4 — State

```bash
terraform state list
terraform state show 'aws_instance.lab7_instances[0]'
```

---

# 🧩 Étape 5 — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>"
```

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | `terraform apply` crée 3 instances | ☐ |
| 2 | Instances nommées `instance-1`, `instance-2`, `instance-3` | ☐ |
| 3 | `instance_count=5` ajoute 2 instances | ☐ |
| 4 | `instance_count=1` supprime 4 instances | ☐ |
| 5 | `terraform destroy` supprime toutes les ressources | ☐ |

---

🎉 **Fin du Lab 7 !**
