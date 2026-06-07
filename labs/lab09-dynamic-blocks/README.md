# 📘 Lab 9 — Dynamic Blocks

## 🎯 Objectifs du Lab

À la fin de ce lab, vous aurez :

- Utilisé les **dynamic blocks** pour créer des règles de Security Group dynamiquement
- Appliqué le principe **DRY** sur les attributs de type block
- Créé des règles **SSH**, **HTTP** et **HTTPS** avec un seul bloc

---

# 🧩 Étape 1 — Créer les fichiers

```bash
cd labs/lab09-dynamic-blocks
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
```

### `locals.tf`

```hcl
locals {
  prefix = "lab09-${var.username}"

  common_tags = {
    Lab       = "lab9"
    Username  = var.username
    ManagedBy = "Terraform"
  }

  # Liste des règles ingress à créer dynamiquement
  sg_rules = [
    { port = 22,  protocol = "tcp", description = "SSH"   },
    { port = 80,  protocol = "tcp", description = "HTTP"  },
    { port = 443, protocol = "tcp", description = "HTTPS" },
  ]
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab9_sg" {
  name        = "${local.prefix}-sg"
  description = "Security group Lab 9 - ${var.username}"

  dynamic "ingress" {
    for_each = local.sg_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ["0.0.0.0/0"]
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow all outbound traffic"
  }

  tags = merge(local.common_tags, {
    Name = "${local.prefix}-sg"
  })
}

resource "aws_instance" "lab9_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab9_sg.id]

  tags = merge(local.common_tags, {
    Name = "${local.prefix}-instance"
  })
}
```

> 💡 Le **dynamic block** itère sur `local.sg_rules` et crée un bloc `ingress` pour chaque élément.
> `ingress.value` donne accès aux attributs de chaque élément de la liste.

### `outputs.tf`

```hcl
output "security_group_id" {
  description = "ID du Security Group"
  value       = aws_security_group.lab9_sg.id
}

output "ingress_rules_count" {
  description = "Nombre de règles ingress créées"
  value       = length(local.sg_rules)
}

output "instance_id" {
  description = "ID de l'instance"
  value       = aws_instance.lab9_instance.id
}
```

---

# 🧩 Étape 2 — Déployer et vérifier les règles

```bash
terraform init
terraform apply -var="username=<votre-prenom>"
```

Vérifiez les règles créées sur le Security Group :

```bash
SG_ID=$(terraform output -raw security_group_id)

aws ec2 describe-security-groups \
  --group-ids $SG_ID \
  --query "SecurityGroups[0].IpPermissions[*].[FromPort,ToPort,IpProtocol]" \
  --output table
```

Résultat attendu :
```
| 22  | 22  | tcp |
| 80  | 80  | tcp |
| 443 | 443 | tcp |
```

---

# 🧩 Étape 3 — Ajouter une règle dynamiquement

Dans `locals.tf`, ajoutez une règle **HTTPS alt** à la liste :

```hcl
sg_rules = [
  { port = 22,   protocol = "tcp", description = "SSH"       },
  { port = 80,   protocol = "tcp", description = "HTTP"      },
  { port = 443,  protocol = "tcp", description = "HTTPS"     },
  { port = 8080, protocol = "tcp", description = "HTTP Alt"  },
]
```

```bash
terraform apply -var="username=<votre-prenom>"
```

> 📋 Terraform ajoute uniquement la nouvelle règle sans recréer le Security Group.

---

# 🧩 Étape 4 — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>"
```

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | Dynamic block crée 3 règles ingress (SSH, HTTP, HTTPS) | ☐ |
| 2 | `ingress_rules_count` retourne `3` | ☐ |
| 3 | Règles visibles via `aws ec2 describe-security-groups` | ☐ |
| 4 | Ajout du port 8080 sans recréer le SG | ☐ |
| 5 | `terraform destroy` supprime toutes les ressources | ☐ |

---

🎉 **Fin du Lab 9 — Vos règles de Security Group sont dynamiques !**

> Le Lab 10 introduit les **data sources** pour récupérer des informations existantes sur AWS.
