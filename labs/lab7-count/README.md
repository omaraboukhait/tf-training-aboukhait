# 📘 Lab 7 — Count & Count Index

## 🎯 Objectifs du Lab

À la fin de ce lab, vous aurez :

- Utilisé **count** pour créer plusieurs instances d'une ressource
- Utilisé **count.index** pour différencier les ressources
- Respecté le principe **DRY** (Don't Repeat Yourself)

---

# 🧩 Étape 1 — Créer les fichiers

```bash
cd labs/lab7-count
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

> ⚠️ Remplacez `<votre-prenom>` par votre prénom. Ex : `tf-training-alice-982908300187`

### `versions.tf`

```hcl
terraform {
  required_version = ">= 1.15.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### `provider.tf`

```hcl
provider "aws" {
  region = var.region
}
```

### `variables.tf`

```hcl
variable "username" {
  description = "Votre prenom"
  type        = string
}

variable "region" {
  description = "Region AWS"
  type        = string
  default     = "eu-west-1"
}

variable "instance_count" {
  description = "Nombre d'instances a créer"
  type        = number
  default     = 3

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 5
    error_message = "Le nombre d'instances doit être entre 1 et 5."
  }
}
```

### `locals.tf`

```hcl
locals {
  prefix = "lab7-${var.username}"

  common_tags = {
    Lab       = "lab7"
    Username  = var.username
    ManagedBy = "Terraform"
  }
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab7_sg" {
  name        = "${local.prefix}-sg"
  description = "Security group Lab 7 - ${var.username}"

  tags = merge(local.common_tags, {
    Name = "${local.prefix}-sg"
  })
}

resource "aws_instance" "lab7_instances" {
  count = var.instance_count

  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab7_sg.id]

  tags = merge(local.common_tags, {
    Name  = "${local.prefix}-instance-${count.index + 1}"
    Index = count.index
  })
}
```

> 💡 `count.index` commence à **0**. On utilise `count.index + 1` pour nommer les instances de 1 à N.
> Terraform crée les ressources en parallèle — `aws_instance.lab7_instances[0]`, `aws_instance.lab7_instances[1]`, etc.

### `outputs.tf`

```hcl
output "instance_ids" {
  description = "IDs de toutes les instances"
  value       = aws_instance.lab7_instances[*].id
}

output "instance_names" {
  description = "Noms de toutes les instances"
  value       = aws_instance.lab7_instances[*].tags.Name
}

output "instance_count" {
  description = "Nombre d'instances créées"
  value       = length(aws_instance.lab7_instances)
}
```

> 💡 `[*]` est le **splat operator** — il récupère un attribut sur toutes les instances du count.

---

# 🧩 Étape 2 — Déployer 3 instances

```bash
terraform init
terraform apply -var="username=<votre-prenom>"
```

Vérifiez les outputs :

```bash
terraform output instance_ids
terraform output instance_names
terraform output instance_count
```

---

# 🧩 Étape 3 — Modifier le count dynamiquement

Augmentez à 5 instances sans modifier le code :

```bash
terraform apply -var="username=<votre-prenom>" -var="instance_count=5"
```

> 📋 Terraform ajoute **uniquement** les 2 instances manquantes — il ne recrée pas les 3 existantes.

Réduisez à 1 instance :

```bash
terraform apply -var="username=<votre-prenom>" -var="instance_count=1"
```

> 📋 Terraform détruit les instances 2, 3, 4 et 5 — il garde uniquement l'instance 0.

---

# 🧩 Étape 4 — Référencer une instance spécifique

```bash
# Accéder à la première instance
terraform state show 'aws_instance.lab7_instances[0]'

# Lister toutes les instances dans le state
terraform state list
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
| 1 | `terraform apply` crée 3 instances avec `count=3` | ☐ |
| 2 | Les instances sont nommées `instance-1`, `instance-2`, `instance-3` | ☐ |
| 3 | `terraform output instance_ids` retourne une liste de 3 IDs | ☐ |
| 4 | `terraform apply -var="instance_count=5"` ajoute 2 instances | ☐ |
| 5 | `terraform apply -var="instance_count=1"` supprime 4 instances | ☐ |
| 6 | `terraform state list` montre les instances indexées `[0]`, `[1]`... | ☐ |
| 7 | `terraform destroy` supprime toutes les ressources | ☐ |

---

# 🛠️ Résolution des problèmes courants

| Problème | Cause | Solution |
|----------|-------|----------|
| `Index out of range` | Référence à un index inexistant | Vérifier `terraform state list` |
| Toutes les instances recréées | Changement d'index en milieu de liste | Comportement normal avec `count` |

---

🎉 **Fin du Lab 7 — Vous savez itérer sur les ressources avec count !**

> Le Lab 8 introduit les **expressions conditionnelles** pour adapter l'infrastructure selon l'environnement.
