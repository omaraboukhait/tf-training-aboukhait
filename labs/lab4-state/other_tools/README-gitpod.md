# 📘 Lab 4 — Terraform Core Behaviour & Remote State (GitPod)

## 🎯 Objectifs du Lab

- Créer votre **bucket S3 avec Terraform** (bootstrap)
- Configurer le **backend S3** avec `use_lockfile = true`
- Observer le comportement Terraform face aux modifications **hors Terraform** et **dans le code**

---

# 🧩 Partie A — Bootstrap : Créer votre bucket S3 avec Terraform

Reprenez votre environnement GitPod → **https://gitpod.io/workspaces**

```bash
mkdir -p labs/bootstrap && cd labs/bootstrap
```

Créez les fichiers dans l'éditeur VS Code GitPod :

### `versions.tf`

```hcl
terraform {
  required_version = ">= 1.15.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
provider "aws" { region = "eu-west-1" }
```

### `variables.tf`

```hcl
variable "username" {
  description = "Votre prenom"
  type        = string
}
```

### `main.tf`

```hcl
data "aws_caller_identity" "current" {}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "tf-training-${var.username}-${data.aws_caller_identity.current.account_id}"
  tags   = { Name = "tf-training-${var.username}", ManagedBy = "Terraform", Username = var.username }
}

resource "aws_s3_bucket_versioning" "state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "state" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default { sse_algorithm = "AES256" }
  }
}

resource "aws_s3_bucket_public_access_block" "state" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### `outputs.tf`

```hcl
output "bucket_name" {
  value = aws_s3_bucket.terraform_state.bucket
}
```

```bash
terraform init
terraform apply -var="username=<votre-prenom>"

BUCKET_NAME=$(terraform output -raw bucket_name)
echo "Mon bucket S3 : $BUCKET_NAME" >> $HOME/tf-training-info.txt
cat $HOME/tf-training-info.txt
```

> ⚠️ Ne supprimez **pas** ce bucket — il sera réutilisé dans tous les labs suivants !

---

# 🧩 Partie B — Créer les fichiers du lab

```bash
cd labs/lab4-state
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
provider "aws" { region = "eu-west-1" }
```

### `variables.tf`

```hcl
variable "username" {
  description = "Votre prenom"
  type        = string
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab4_sg" {
  name        = "lab4-sg-${var.username}"
  description = "Security group Lab 4 - ${var.username}"
  tags = { Name = "lab4-sg-${var.username}", Lab = "lab4", Username = var.username }
}

resource "aws_instance" "lab4_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab4_sg.id]
  tags = { Name = "lab4-instance-${var.username}", Lab = "lab4", Username = var.username }
}
```

### `outputs.tf`

```hcl
output "instance_id" { value = aws_instance.lab4_instance.id }
output "security_group_id" { value = aws_security_group.lab4_sg.id }
```

---

# 🧩 Partie C — Init avec Backend S3

```bash
BUCKET_NAME=$(grep "bucket" $HOME/tf-training-info.txt | awk '{print $NF}')

terraform init \
  -backend-config="bucket=${BUCKET_NAME}" \
  -backend-config="key=lab4/terraform.tfstate"

terraform apply -var="username=<votre-prenom>"
aws s3 ls s3://${BUCKET_NAME}/lab4/
```

---

# 🧩 Partie D — Modifier hors Terraform

```bash
INSTANCE_ID=$(terraform output -raw instance_id)
aws ec2 create-tags --resources $INSTANCE_ID --tags Key=ModifiedOutside,Value=true
terraform plan -var="username=<votre-prenom>"
terraform refresh -var="username=<votre-prenom>"
terraform state show aws_instance.lab4_instance | grep ModifiedOutside
```

---

# 🧩 Partie E — Modifier dans le code

Dans l'éditeur VS Code GitPod, changez `t2.micro` en `t2.small` dans `main.tf` :

```bash
terraform plan -var="username=<votre-prenom>"
```

Remettez `t2.micro` puis vérifiez :

```bash
terraform plan -var="username=<votre-prenom>"
```

Résultat attendu : `No changes.`

---

# 🧩 Partie F — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>"
```

> ⚠️ Ne détruisez **pas** le bootstrap !

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | Bucket S3 créé **avec Terraform** via le bootstrap | ☐ |
| 2 | `terraform init -backend-config` réussi | ☐ |
| 3 | State visible dans S3 après `terraform apply` | ☐ |
| 4 | `terraform plan` détecte la modification hors Terraform | ☐ |
| 5 | `terraform plan` détecte le changement `t2.small` | ☐ |
| 6 | `terraform destroy` supprime les ressources du lab | ☐ |

---

🎉 **Fin du Lab 4 — Remote state S3 configuré pour toute la formation !**
