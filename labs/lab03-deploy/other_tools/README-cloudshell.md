# 📘 Lab 3 — State Management (AWS CloudShell)

## 🎯 Objectifs du Lab

- Explorer le **fichier d'état Terraform**
- Lister, inspecter et renommer des ressources avec `terraform state`
- Comprendre **desired state** vs **current state**

---

# 🧩 Étape 1 — Préparer l'environnement

Ouvrez CloudShell et réinstallez Terraform si nécessaire :

```bash
bash $HOME/install-terraform.sh
cd $HOME/terraform-training/lab03-state
```

Créez les fichiers :

```bash
cat > versions.tf << 'EOF'
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
provider "aws" { region = "eu-west-1" }
EOF

cat > variables.tf << 'EOF'
variable "username" {
  description = "Votre prenom"
  type        = string
}
EOF

cat > main.tf << 'EOF'
resource "aws_security_group" "lab3_sg" {
  name        = "lab3-sg-${var.username}"
  description = "Security group Lab 3 - ${var.username}"
  tags = {
    Name     = "lab3-sg-${var.username}"
    Lab      = "lab3"
    Username = var.username
  }
}

resource "aws_instance" "lab3_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab3_sg.id]
  tags = {
    Name     = "lab3-instance-${var.username}"
    Lab      = "lab3"
    Username = var.username
  }
}
EOF

cat > outputs.tf << 'EOF'
output "instance_id" {
  value = aws_instance.lab3_instance.id
}
EOF
```

Déployez :

```bash
terraform init
terraform apply -var="username=<votre-prenom>"
```

---

# 🧩 Étape 2 — State list & show

```bash
terraform state list
terraform state show aws_instance.lab3_instance
terraform state show aws_security_group.lab3_sg
```

---

# 🧩 Étape 3 — Renommer dans le State

```bash
terraform state mv aws_instance.lab3_instance aws_instance.web_server
terraform state list
sed -i 's/lab3_instance/web_server/g' main.tf outputs.tf
terraform plan -var="username=<votre-prenom>"
```

Résultat attendu : `No changes.`

---

# 🧩 Étape 4 — Desired vs Current State

```bash
sed -i 's/t2.micro/t2.small/' main.tf
terraform plan -var="username=<votre-prenom>"
sed -i 's/t2.small/t2.micro/' main.tf
```

---

# 🧩 Étape 5 — Refresh

```bash
INSTANCE_ID=$(terraform output -raw instance_id)
aws ec2 create-tags --resources $INSTANCE_ID --tags Key=ModifiedOutside,Value=true
terraform refresh -var="username=<votre-prenom>"
terraform state show aws_instance.web_server | grep ModifiedOutside
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
| 1 | `terraform state list` liste les 2 ressources | ☐ |
| 2 | `terraform state mv` renomme sans recréer | ☐ |
| 3 | `terraform plan` retourne "No changes" après renommage | ☐ |
| 4 | `terraform plan` détecte le changement `t2.small` | ☐ |
| 5 | `terraform refresh` capture le tag ajouté manuellement | ☐ |
| 6 | `terraform destroy` supprime toutes les ressources | ☐ |

---

🎉 **Fin du Lab 3 !**
