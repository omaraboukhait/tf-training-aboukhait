# 📘 Lab 5 — Outputs & Variables (AWS CloudShell)

## 🎯 Objectifs du Lab

- Organiser le code selon les **bonnes pratiques** Terraform
- Exporter des attributs avec les **outputs**
- Assigner des variables via **CLI**, **tfvars** et **TF_VAR_**

---

# 🧩 Étape 1 — Créer les fichiers

```bash
bash $HOME/install-terraform.sh
cd $HOME/terraform-training/lab5-outputs
```

```bash
cat > backend.tf << 'EOF'
terraform {
  backend "s3" {
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }
}
EOF

cat > versions.tf << 'EOF'
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
EOF

cat > provider.tf << 'EOF'
provider "aws" { region = var.region }
EOF

cat > variables.tf << 'EOF'
variable "username" {
  description = "Votre prenom"
  type        = string
  validation {
    condition     = length(var.username) >= 2 && length(var.username) <= 20
    error_message = "Le username doit contenir entre 2 et 20 caractères."
  }
}
variable "region" { type = string; default = "eu-west-1" }
variable "instance_type" {
  type    = string
  default = "t2.micro"
  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Le type d'instance doit être t2.micro, t2.small ou t2.medium."
  }
}
variable "environment" {
  type    = string
  default = "dev"
  validation {
    condition     = contains(["dev", "prod"], var.environment)
    error_message = "L'environnement doit être dev ou prod."
  }
}
EOF

cat > main.tf << 'EOF'
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
EOF

cat > outputs.tf << 'EOF'
output "instance_id" { description = "ID de l'instance EC2" ; value = aws_instance.lab5_instance.id }
output "instance_public_ip" { description = "IP publique" ; value = aws_instance.lab5_instance.public_ip }
output "security_group_id" { description = "ID du SG" ; value = aws_security_group.lab5_sg.id }
output "security_group_name" { description = "Nom du SG" ; value = aws_security_group.lab5_sg.name }
output "deployment_summary" {
  description = "Résumé du déploiement"
  value = { username = var.username, environment = var.environment, instance = aws_instance.lab5_instance.id, region = var.region }
}
EOF
```

---

# 🧩 Étape 2 — Les 3 façons d'assigner les variables

## Méthode 1 — CLI flag

```bash
terraform init
terraform apply -var="username=<votre-prenom>" -var="environment=dev"
```

## Méthode 2 — tfvars

```bash
cat > terraform.tfvars << 'EOF'
username      = "<votre-prenom>"
environment   = "dev"
instance_type = "t2.micro"
EOF

terraform apply
```

## Méthode 3 — TF_VAR_

```bash
export TF_VAR_username="<votre-prenom>"
export TF_VAR_environment="dev"
terraform apply
```

---

# 🧩 Étape 3 — Tester la validation

```bash
terraform plan -var="username=<votre-prenom>" -var="environment=staging"
terraform plan -var="username=<votre-prenom>" -var="instance_type=t3.large"
```

---

# 🧩 Étape 4 — Utiliser terraform output

```bash
terraform output
terraform output instance_id
terraform output -json deployment_summary
terraform output -raw instance_id
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
| 1 | 6 fichiers séparés créés | ☐ |
| 2 | Validation bloque `environment=staging` | ☐ |
| 3 | Les 3 méthodes d'assignation fonctionnent | ☐ |
| 4 | `terraform output -json` affiche un objet JSON | ☐ |
| 5 | `terraform destroy` supprime les ressources | ☐ |

---

🎉 **Fin du Lab 5 !**
